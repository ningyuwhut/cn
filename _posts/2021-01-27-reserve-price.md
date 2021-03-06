---
  layout: post
  title: 保留价
  categories: MachineLearning
  tags:
--- 

PID算法是自动控制领域的经典算法。现在在互联网领域也有很多应用，比如用于成本控制，用于预算平滑，用于智能出价。


在成本控制中，比如是cpc广告，目标cpc是1块钱，那么PID可以以当前的实际cpc为输入，输出是$\lambda$，可以使用下面的公式对出价进行调整(假设使用cpm进行结算）。

$$
ecpm = (cpc + \lambda) * ctr * 1000
$$

在预算平滑中，也可以使用PID算法。

预算平滑有两种方式，一种是节流，控制参竞率；一种是预算调整，其实调整的是出价。

在<Feedback Control of Real-Time Display Advertising>也是采用PID进行预算平滑。


参考:

https://zhuanlan.zhihu.com/p/39573490


https://zhuanlan.zhihu.com/p/139244173


https://www.infoq.cn/article/akkwpvsnium9tmhhuu3f


https://zhuanlan.zhihu.com/p/140933417

https://mp.weixin.qq.com/s?src=11&timestamp=1611653995&ver=2852&signature=K9jmEm2*PeooBl-fFczZs3UYIjF3kqM5U9jHeUQvuLdYbjmaJ-GI-33jyNg5OmwJjlcCLifhcz7IX51g60*5qdKoO9OVfvlJhFdrlvhKcqiPWtAO7GMs*Fbq6zAj1dk7&new=1

https://mp.weixin.qq.com/s?src=11&timestamp=1611653995&ver=2852&signature=KoZKsJRcrkhHFhqXC0H1pswBtm*sd5*hf3T7dXKAIHGr1XBKE3i2G07la9AJyZWPwDoZsp84N1BEi37Sfbqd*VmIaqLIEBN1GMQKoni8FNFpb8fnPqRSTYQH*7twY05Y&new=1


https://zhuanlan.zhihu.com/p/98707378