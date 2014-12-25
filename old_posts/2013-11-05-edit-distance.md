---
layout: post 
title: LeetCode Generate Parentheses
categories:
- Online Judge
tags:
- LeetCode
- interview
---

>Given two words word1 and word2, find the minimum number of steps required to convert word1 to word2. (each operation is counted as 1 step.)
>
>You have the following 3 operations permitted on a word:
>
>a) Insert a character
>b) Delete a character
>c) Replace a character

思路:

动态规划

代码:

    int minDistance(string word1, string word2) {
     
        int len1 = word1.size();
        int len2 = word2.size();
        
        if( len1 == 0 )
            return len2;
        if( len2 == 0 )
            return len1;
        
        int **distance = new int*[len1+1];
        for( int i = 0; i <= len1; ++i)
            distance[i] = new int[len2+1];
        
        for( int i =0; i <= len1; ++i )
            distance[i][0] = i;
        
        for( int i =0; i <= len2; ++i )
            distance[0][i] = i;
            
        for( int i = 1; i <= len1; ++i ){
            for( int j = 1; j <= len2; ++j ){
                int temp;
                if( word1[i-1] == word2[j-1] )
                    temp = 0;
                else
                    temp = 1;
              
                int a = distance[i-1][j] + 1;
                int b = distance[i][j-1] + 1;
                int c = distance[i-1][j-1] + temp;
                int min;
                if( a > b )
                    min = b;
                else
                    min = a;
                if( c < min )
                    min = c;
                
                distance[i][j] = min;
            }
        }
       int result = distance[len1][len2];

       for( int i = 0; i <= len1; ++i )
           delete[] distance[i];
       delete[] distance;
       
       return result;
    }
