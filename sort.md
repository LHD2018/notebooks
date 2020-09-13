# 排序算法

![](https://i.loli.net/2020/09/08/5AzZ7V8qg1ujhOf.png)


## 冒泡排序

+ 每一轮循环比较相邻元素，把最大（最小）的数冒出

~~~c++
void bubbleSort(vector<int> &v){
    int len = v.size();
    for (int i = 0; i < len; i++){
        for(int j = 0;j < len - 1 - i;j++){
            if(v[j] > v[j+1]){
                swap(v[j],v[j+1]);
            }
        }
    }  
}
// 优化后

~~~



