---
  layout: post
  title: xgboost 代码阅读
  categories: MachineLearning
  tags:
--- 

xgboost的入口文件是cli_main.cc

````
int main(int argc, char *argv[]) {
  try {
    xgboost::CLI cli(argc, argv);
    return cli.Run();
  } catch (dmlc::Error const& e) {
    // This captures only the initialization error.
    xgboost::CLIError(e);
    return 1;
  }
  return 0;
}
````

````
int Run() {
    switch (this->print_info_) {
    case kNone:
      break;
    case kVersion: {
      this->PrintVersion();
      return 0;
    }
    case kHelp: {
      this->PrintHelp();
      return 0;
    }
    }

    try {
      switch (param_.task) {
      case kTrain:
        CLITrain();
        break;
      case kDumpModel:
        CLIDumpModel();
        break;
      case kPredict:
        CLIPredict();
        break;
      }
    } catch (dmlc::Error const& e) {
      xgboost::CLIError(e);
      return 1;
    }
    return 0;
  }
````

包含三个任务，训练、评估和导出模型。

UpdateOneIter 负责训练 ,EvalOneIter 负责评估。

````
void UpdateOneIter(int iter, std::shared_ptr<DMatrix> train) override {
    monitor_.Start("UpdateOneIter");
    TrainingObserver::Instance().Update(iter);
    this->Configure();
    if (generic_parameters_.seed_per_iteration || rabit::IsDistributed()) {
      common::GlobalRandom().seed(generic_parameters_.seed * kRandSeedMagic + iter);
    }
    this->CheckDataSplitMode();
    this->ValidateDMatrix(train.get());

    auto& predt = this->cache_.Cache(train, generic_parameters_.gpu_id);

    monitor_.Start("PredictRaw");
    this->PredictRaw(train.get(), &predt, true);
    TrainingObserver::Instance().Observe(predt.predictions, "Predictions");
    monitor_.Stop("PredictRaw");

    monitor_.Start("GetGradient");
    obj_->GetGradient(predt.predictions, train->Info(), iter, &gpair_);
    monitor_.Stop("GetGradient");
    TrainingObserver::Instance().Observe(gpair_, "Gradients");

    gbm_->DoBoost(train.get(), &gpair_, &predt);
    monitor_.Stop("UpdateOneIter");
  }
````

PredictRaw 先进行预测，GetGradient根据预测结果计算梯度，DoBoost执行一次建树，更新模型。

````
void GBTree::DoBoost(DMatrix* p_fmat,
                     HostDeviceVector<GradientPair>* in_gpair,
                     PredictionCacheEntry* predt) {
  std::vector<std::vector<std::unique_ptr<RegTree> > > new_trees;
  const int ngroup = model_.learner_model_param->num_output_group;
  ConfigureWithKnownData(this->cfg_, p_fmat);
  monitor_.Start("BoostNewTrees");
  CHECK_NE(ngroup, 0);
  if (ngroup == 1) {
    std::vector<std::unique_ptr<RegTree> > ret;
    BoostNewTrees(in_gpair, p_fmat, 0, &ret);
    new_trees.push_back(std::move(ret));
  } else {
    CHECK_EQ(in_gpair->Size() % ngroup, 0U)
        << "must have exactly ngroup * nrow gpairs";
    // TODO(canonizer): perform this on GPU if HostDeviceVector has device set.
    HostDeviceVector<GradientPair> tmp(in_gpair->Size() / ngroup,
                                       GradientPair(),
                                       in_gpair->DeviceIdx());
    const auto& gpair_h = in_gpair->ConstHostVector();
    auto nsize = static_cast<bst_omp_uint>(tmp.Size());
    for (int gid = 0; gid < ngroup; ++gid) {
      std::vector<GradientPair>& tmp_h = tmp.HostVector();
#pragma omp parallel for schedule(static)
      for (bst_omp_uint i = 0; i < nsize; ++i) {
        tmp_h[i] = gpair_h[i * ngroup + gid];
      }
      std::vector<std::unique_ptr<RegTree> > ret;
      BoostNewTrees(&tmp, p_fmat, gid, &ret);
      new_trees.push_back(std::move(ret));
    }
  }
  monitor_.Stop("BoostNewTrees");
  this->CommitModel(std::move(new_trees), p_fmat, predt);
}
````

DoBoost 通过BoostNewTrees来进行模型迭代，生成新的决策树。
最后通过update函数来执行树的生成操作。

````
void Update(HostDeviceVector<GradientPair> *gpair,
              DMatrix* dmat,
              const std::vector<RegTree*> &trees) override {
    if (rabit::IsDistributed()) {
      LOG(FATAL) << "Updater `grow_colmaker` or `exact` tree method doesn't "
                    "support distributed training.";
    }
    this->LazyGetColumnDensity(dmat);
    // rescale learning rate according to size of trees
    float lr = param_.learning_rate;
    param_.learning_rate = lr / trees.size();
    interaction_constraints_.Configure(param_, dmat->Info().num_row_);
    // build tree
    for (auto tree : trees) {
      Builder builder(
        param_,
        colmaker_param_,
        std::unique_ptr<SplitEvaluator>(spliteval_->GetHostClone()),
        interaction_constraints_, column_densities_);
      builder.Update(gpair->ConstHostVector(), dmat, tree);
    }
    param_.learning_rate = lr;
  }
````

````
 // update one tree, growing
    virtual void Update(const std::vector<GradientPair>& gpair,
                        DMatrix* p_fmat,
                        RegTree* p_tree) {
      std::vector<int> newnodes;
      this->InitData(gpair, *p_fmat, *p_tree);
      this->InitNewNode(qexpand_, gpair, *p_fmat, *p_tree);
      for (int depth = 0; depth < param_.max_depth; ++depth) {
        this->FindSplit(depth, qexpand_, gpair, p_fmat, p_tree);
        this->ResetPosition(qexpand_, p_fmat, *p_tree);
        this->UpdateQueueExpand(*p_tree, qexpand_, &newnodes);
        this->InitNewNode(newnodes, gpair, *p_fmat, *p_tree);
        for (auto nid : qexpand_) {
          if ((*p_tree)[nid].IsLeaf()) {
            continue;
          }
          int cleft = (*p_tree)[nid].LeftChild();
          int cright = (*p_tree)[nid].RightChild();
          spliteval_->AddSplit(nid,
                               cleft,
                               cright,
                               snode_[nid].best.SplitIndex(),
                               snode_[cleft].weight,
                               snode_[cright].weight);
          interaction_constraints_.Split(nid, snode_[nid].best.SplitIndex(), cleft, cright);
        }
        qexpand_ = newnodes;
        // if nothing left to be expand, break
        if (qexpand_.size() == 0) break;
      }
      // set all the rest expanding nodes to leaf
      for (const int nid : qexpand_) {
        (*p_tree)[nid].SetLeaf(snode_[nid].weight * param_.learning_rate);
      }
      // remember auxiliary statistics in the tree node
      for (int nid = 0; nid < p_tree->param.num_nodes; ++nid) {
        p_tree->Stat(nid).loss_chg = snode_[nid].best.loss_chg;
        p_tree->Stat(nid).base_weight = snode_[nid].weight;
        p_tree->Stat(nid).sum_hess = static_cast<float>(snode_[nid].stats.sum_hess);
      }
    }
````

在InitNewNode中新建节点，并执行一些初始化操作，包括计算该节点下的样本的梯度和二阶梯度，并据此计算节点的权重（叶子输出值）和目标函数值。

````
 /*!
     * \brief initialize the base_weight, root_gain,
     *  and NodeEntry for all the new nodes in qexpand
     */
    inline void InitNewNode(const std::vector<int>& qexpand,
                            const std::vector<GradientPair>& gpair,
                            const DMatrix& fmat,
                            const RegTree& tree) {
      {
        // setup statistics space for each tree node
        for (auto& i : stemp_) {
          i.resize(tree.param.num_nodes, ThreadEntry());
        }
        snode_.resize(tree.param.num_nodes, NodeEntry());
      }
      const MetaInfo& info = fmat.Info();
      // setup position
      const auto ndata = static_cast<bst_omp_uint>(info.num_row_);
      #pragma omp parallel for schedule(static)
      for (bst_omp_uint ridx = 0; ridx < ndata; ++ridx) {
        const int tid = omp_get_thread_num();
        if (position_[ridx] < 0) continue;
        stemp_[tid][position_[ridx]].stats.Add(gpair[ridx]);
      }
      // sum the per thread statistics together
      for (int nid : qexpand) {
        GradStats stats;
        for (auto& s : stemp_) {
          stats.Add(s[nid].stats);
        }
        // update node statistics
        snode_[nid].stats = stats;
      }
      // calculating the weights
      for (int nid : qexpand) {
        bst_uint parentid = tree[nid].Parent();
        snode_[nid].weight = static_cast<float>(
            spliteval_->ComputeWeight(parentid, snode_[nid].stats));
        snode_[nid].root_gain = static_cast<float>(
            spliteval_->ComputeScore(parentid, snode_[nid].stats, snode_[nid].weight));
      }
    }
  ````

  然后计算最优分裂特征和分裂点

  ````
  // find splits at current level, do split per level
    inline void FindSplit(int depth,
                          const std::vector<int> &qexpand,
                          const std::vector<GradientPair> &gpair,
                          DMatrix *p_fmat,
                          RegTree *p_tree) {
      auto feat_set = column_sampler_.GetFeatureSet(depth);
      for (const auto &batch : p_fmat->GetBatches<SortedCSCPage>()) {
        this->UpdateSolution(batch, feat_set->HostVector(), gpair, p_fmat);
      }
      // after this each thread's stemp will get the best candidates, aggregate results
      this->SyncBestSolution(qexpand);
      // get the best result, we can synchronize the solution
      for (int nid : qexpand) {
        NodeEntry const &e = snode_[nid];
        // now we know the solution in snode[nid], set split
        if (e.best.loss_chg > kRtEps) {
          bst_float left_leaf_weight =
              spliteval_->ComputeWeight(nid, e.best.left_sum) *
              param_.learning_rate;
          bst_float right_leaf_weight =
              spliteval_->ComputeWeight(nid, e.best.right_sum) *
              param_.learning_rate;
          p_tree->ExpandNode(nid, e.best.SplitIndex(), e.best.split_value,
                             e.best.DefaultLeft(), e.weight, left_leaf_weight,
                             right_leaf_weight, e.best.loss_chg,
                             e.stats.sum_hess,
                             e.best.left_sum.GetHess(), e.best.right_sum.GetHess(),
                             0);
        } else {
          (*p_tree)[nid].SetLeaf(e.weight * param_.learning_rate);
        }
      }
    }
  ````

  计算完之后将节点划分到各自的子节点中

  ````
  // reset position of each data points after split is created in the tree
    inline void ResetPosition(const std::vector<int> &qexpand,
                              DMatrix* p_fmat,
                              const RegTree& tree) {
      // set the positions in the nondefault
      this->SetNonDefaultPosition(qexpand, p_fmat, tree);
      // set rest of instances to default position
      // set default direct nodes to default
      // for leaf nodes that are not fresh, mark then to ~nid,
      // so that they are ignored in future statistics collection
      const auto ndata = static_cast<bst_omp_uint>(p_fmat->Info().num_row_);

#pragma omp parallel for schedule(static)
      for (bst_omp_uint ridx = 0; ridx < ndata; ++ridx) {
        CHECK_LT(ridx, position_.size())
            << "ridx exceed bound " << "ridx="<<  ridx << " pos=" << position_.size();
        const int nid = this->DecodePosition(ridx);
        if (tree[nid].IsLeaf()) {
          // mark finish when it is not a fresh leaf
          if (tree[nid].RightChild() == -1) {
            position_[ridx] = ~nid;
          }
        } else {
          // push to default branch
          if (tree[nid].DefaultLeft()) {
            this->SetEncodePosition(ridx, tree[nid].LeftChild());
          } else {
            this->SetEncodePosition(ridx, tree[nid].RightChild());
          }
        }
      }
    }
````

叶子节点和缺省值划分到默认节点。

然后，更新待分裂的节点队列，将新一层的节点加入进来。
