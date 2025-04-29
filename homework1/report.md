# 41243210
# 41243231

作業一

## 解題說明

本題要求測試4種排序方法，在最壞資料情況下測試獲取他們的效率時間範圍，並寫出複合排序函數（Composite Sorting Function），使其能對所有 n 提供最佳性能

### 解題策略
1.實作四種排序：Insertion Sort、Quick Sort、Merge Sort、Heap Sort。

2.測試4種排序他們各自最壞情況下的時間複雜度（n = 30,500, 1000, ..., 5000,10000）。

3.測試平均情況下的時間複雜度（使用亂數產生資料）。

4.比較結果並畫圖。

5.寫出綜合排序函式（Composite Sort Function），再去比較另外4個排序


## 程式實作
所使用到的所有函式庫
```cpp
#include <iostream>
#include <cmath>
#include <fstream>
#include <vector>
#include <ctime>
#include <cstdlib>
#include <string>
#include <algorithm>
```

讀取文字檔案

```cpp
vector<int> read_data(const string& filename, int& n) {
    ifstream in;
    in.open(filename);
    if (!in.is_open()) {
        throw runtime_error("Cannot open file: " + filename);
    }

    // 讀入資料筆數
    in >> n;
    vector<int> a(n);

    // 讀入資料
    for (int i = 0; i < n; i++) {
        if (!(in >> a[i])) {
            in.close();
            throw runtime_error("Invalid data format in file: " + filename);
        }
    }

    in.close();
    return a;
}
```
寫入文字檔案

```cpp
void write_to_file(const vector<int>& data, const string& filename) {
    ofstream out(filename);
    out << data.size() << endl;
    for (int i = 0; i < data.size(); i++) {
        out << data[i];
        if (i < data.size() - 1) out << " ";
    }
    out << endl;
    out.close();
}
```


生成insertion sort最糟糕的資料

```cpp
void generate_insertion_worst(int n) {
    vector<int> data(n);
    for (int i = 0; i < n; i++) {
        data[i] = n - i;
    }
    write_to_file(data, "data.txt");
}
}
```

merge sort和heap sort的前置生成函數


```cpp
void permute(vector<int>& arr, int n) {
    for (int i = n - 1; i >= 1; --i) {
        int j = rand() % (i + 1); // Random index from 0 to i
        swap(arr[i], arr[j]);
    }
}
```
生成merge sort和heap sort最糟糕資料

```cpp
void generate_merge_worst(int n) {
    srand(time(0));
    vector<int> arr(n);

    // Initialize array with 1 to n
    for (int i = 0; i < n; ++i) {
        arr[i] = i + 1;
    }

    // Generate one random permutation
    permute(arr, n);
    write_to_file(arr, "data.txt");
}
```

生成隨機資料

```cpp
void generate_random_data(int n) {
    ofstream out("data.txt");
    if (!out.is_open()) {
        cerr << "無法開啟檔案: " << "data.txt" << endl;
        return;
    }

    srand(time(NULL)); // 設定隨機種子

    out << n << endl; // 寫入資料筆數
    for (int i = 0; i < n; i++) {
        out << rand() % n << " ";
    }
    out << endl;

    out.close();
}
```


Insertion Sort
將一組資料分成兩部分，先設定成一部分是已排序的且只包含第一個元素，另一部分是未排序的。接著，Insertion Sort會從未排序的資料中取出一個元素，並把它插入到已排序的那一部分的正確位置，為了保證這個插入的正確性需要從已排序部分的最後一個元素開始比較，如果這個元素比待插入的值大，則將它向右移動一格，如此重複直到找到合適的位置，並排序直到完成。
```cpp
template<class T>
vector<T> insertsort(vector<T> a, int n){
    T temp;
    for(int i = 1; i < n; i++){
        temp = a[i];
        int j = i - 1;
        while(j >= 0 && temp < a[j]){
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = temp;
    }
    return a;
}
```

Quick Sort
1.找pivot
先從當前數列找一個pivot，由於用的是三數取中值low、mid、high 三者中取中值，再把它交換到 high。
2.根據pivot做比大小
掃描從low到high之間的數字，把小於 pivot 的元素都移到左邊，大於的放到右邊
3.遞迴
對 pivot 左邊跟右邊的數列排序，並且重複這個過程值到元素被排序完成。
```cpp
template<class T>
vector<T> quicksort(vector<T> a, const int& front, const int& end, const bool& worst) {
    if (worst)
        quicksortWorst(a, front, end);
    else
        quicksortNormal(a, front, end);
    return a;
}
```

```cpp
template<class T>
void quicksortNormal(vector<T>& a, int left, int right){
    if(left < right){

        // 取三個數的中間值
        int mid = (left + right) / 2;
        if (a[mid] < a[left]) swap(a[left], a[mid]);
        if (a[right] < a[left]) swap(a[left], a[right]);
        if (a[mid] < a[right]) swap(a[mid], a[right]);
        // 將pivot移到最左邊
        swap(a[left], a[right]);

        int i = left, j = right + 1;
        do{
            do i++; while(a[i] < a[left]);
            do j--; while(a[j] > a[left]);
            if(i < j) swap(a[i], a[j]);
        }while(i < j);
        
        swap(a[left], a[j]);
        
        quicksort(a, left, j - 1);
        quicksort(a, j + 1, right);
    }

}
```

Merge Sort
1.把元素切割成序列
先將資料不斷地切割成越來越小的子序列，1000 500 ..... 1，直到每個子序列只包含一個元素為止
2.合併
兩兩合併序列，1 2 4 ....1000，並且在每次合併時都會以排序的方式將它們合併成一個有序的序列，這個過程會一直合併序列直到所有子序列都被合併成序列。
```cpp
// 將左右陣列合併並排列
template<class T>
void merge(vector<T>& a, const int& front, const int& mid, const int& end){
    int i, j, count;
    i = j = 0;
    count = front;
    vector<T>   left(a.begin()+front, a.begin()+mid+1), 
                right(a.begin()+mid+1, a.begin()+end+1);

    // 若左陣列和右邊界都還沒到底，繼續比較
    while(i < left.size() && j < right.size()){
        if(left[i] < right[j]){
            a[count] = left[i];
            i++;
        }
        else{
            a[count] = right[j];
            j++;
        }
        count++;
    }

    // 將剩下的數值填入結果
    while(i < left.size())
        a[count++] = left[i++];
    while(j < right.size())
        a[count++] = right[j++];

}

template<class T>
void mergesort(const vector<T>& a, const int& front, const int& end){
    // 只剩一個元素時，回傳單一元素的vector
    if (front >= end) {
        vector<T> single = { a[front] };
        return single;
    }

    int mid = (front + end) / 2;
    vector<T> left = mergesort(a, front, mid);
    vector<T> right = mergesort(a, mid + 1, end);
    return merge(front, end);
}
```

Heap Sort
1.建立最大堆，將原始陣列重組成最大堆積結構，向前對每個節點進行比大小的操作，如果出現錯誤則是進行交換，確保所有子樹都符合最大堆的條件。
2.排序階段，會一直重複如下操作
將第一個值與「最後一個節點」交換位置，這樣最大值就被放到了陣列尾端，然後把最後一個節點視為排序成功，之後重複進行最大堆
```cpp
template<class T>
void maxheapify(vector<T>& a, const int& root, const int& len){
    int left = 2*root+1;
    int right = 2*root+2;
    int largest = root;

    if(left < len && a[left] >= a[largest]){
        largest = left;
    }
    
    if(right < len && a[right] >= a[largest]){
        largest = right;
    }

    if(largest != root){
        swap(a[root], a[largest]);
        maxheapify(a, largest, len);
    }
}

// 建立最大堆積樹
template<class T>
void maxheap(vector<T>& a){
    for(int i = a.size()/2; i >= 0; i--){
        maxheapify(a, i, a.size());
    }
}

template<class T>
void heapsort(vector<T> a){
    int len = a.size();
    maxheap(a);
    for(int i = a.size()-1; i > 0; i--){
        swap(a[i], a[0]);
        // 減掉以排序的長度在找下個最大堆積
        maxheapify(a, 0, --len);
    }
    return a;
}
```

Composite sort
```c++
template<class T>
void compositesort(vector<T>& a, const int& left, const int& right, int depth) {
    int size = right - left + 1;

    if (left >= right) return;

    // 資料筆數低時使用InsertSort
    if (size <= 30) {
        a = insertsort(a, left, right);
        return;
    }

    // 遞迴深度高時HeapSort
    if (depth >= log2(a.size())) {
        a = heapsort(a, left, right);
        return;
    }

    // 預設使用Quicksort
    
    // 取三個數的中間值
    int mid = (left + right) / 2;
    if (a[mid] < a[left]) swap(a[left], a[mid]);
    if (a[right] < a[left]) swap(a[left], a[right]);
    if (a[mid] < a[right]) swap(a[mid], a[right]);
    swap(a[left], a[right]);

    int i = left, j = right + 1;
    do {
        do i++; while (a[i] < a[left]);
        do j--; while (a[j] > a[left]);
        if (i < j) swap(a[i], a[j]);
    } while (i < j);

    swap(a[left], a[j]);

    compositesort(a, left, j - 1, depth+1);
    compositesort(a, j + 1, right, depth+1);
}
```

## 效能分析

### 不同排序的空間複雜度
| 排序演算法 | 空間複雜度| 
|----------|--------------|
| Insertion Sort   | O($1$) |
| Merge Sort   | O($n$)|
| Heap Sort   | O($1$) |
| Quick Sort   | O($log n$)



### 不同排序的時間複雜度
| 排序演算法 | Best| Worst| Avg| 
|----------|--------------|--------------|--------------|
| Insertion Sort   | O($n$)  | O($n^2$) | O($n^2$) |
| Merge Sort   | O($n log n$)  | O($n log n$) | O($n log n$) |
| Heap Sort   | O($n log n$)   | O($n log n$) | O($n log n$) |
| Quick Sort   | O($n log n$)  | O($n^2$) | O($n log n$)




### 不同排序運行效率最佳範圍
| 排序演算法 | 最佳效率範圍| 理由 |
|----------|--------------|----------|
| Insertion Sort   | $n <= 30$      | 小數據處理快速、記憶體占用小        | 
| Merge Sort   | $n >= 500$     | 各種情況時間複雜度都為O(nlogn)        | 
| Heap Sort   | $n >= 500$       | 記憶體占用小      | 
| Quick Sort   | $n >= 500$      | 平均最快       | 


## 測試與驗證

測試方式:
更新最大記憶體函式
```c++
void update_max_memory(size_t memory) {
    if (memory > max_memory_usage) {
        max_memory_usage = memory;
    }
}
```
對每個排序使用不同的測量方式
Insertion sort 只有新增一個暫存變數
```c++
update_max_memory(sizeof(temp));
```
Quick sort 看遞迴深度，每次有i,j,mid,piovt存
```c++
update_max_memory(depth * 4 * sizeof(int));
```
Merge sort 額外空間儲存左右陣列
```c++
size_t current_memory = memory_of_vector(left) + memory_of_vector(right);
update_max_memory(current_memory);
```
Heap sort 只有left、right、largest
```c++
update_max_memory(3 * sizeof(int)); 
```
### 計時方式
使用在<ctime>中的clock()，單位為毫秒
用法如以下程式
```c++
start = clock();
insertsort(a, n);
stop = clock();
(stop - start) / CLOCKS_PER_SEC // 轉成秒
```

### 時間精確度:s
### 不同排序最糟情況運行時間
| 測試案例 | 輸入參數 $n$ | Insertion Sort(55次平均) | Quick Sort(55次平均) | Merge Sort(20組取最大)|Heap Sort(20組取最大) |
|----------|--------------|----------|----------|----------|----------|
| 測試一   | $n = 30$      | 0       | 0       |0.002        |0  |
| 測試二   | $n = 500$      | 0.000618182        | 0.0000909     |0.002        |0.001 |
| 測試三   | $n = 1000$      | 0.00238182       | 0.000218182       |0.002       |0.001       |
| 測試四   | $n = 2000$      | 0.00958182      | 0.0034     |0.003        |0.001    |
| 測試五   | $n = 3000$     | 0.0218182 |0.00754545 | 0.004  |   0.001    |
| 測試六   | $n = 4000$     |  0.0392182|0.0133818|  0.005    |0.002     |
| 測試七   | $n = 5000$     |  0.0600909|0.0208545 |  0.006    |0.003     |
| 測試八   | $n = 10000$     |  0.238436|0.0834545 |  0.012    |0.004     |



![糟糕狀況折線圖](https://cdn.discordapp.com/attachments/930060410823016509/1366049723403735100/QtXVX4G8N9wAAAABJRU5ErkJggg.png?ex=680f8872&is=680e36f2&hm=c537541d964d58b2a77f98ea6fe97b23d203445aa6a92c8de89b9746891e0cf8&)
根據以上測出來的資料可以看出Insertion Sort符合最壞情況(O($n^2$) Quick Sort符合最壞情況(O($n^2$) Merge Sort符合最壞情況O($n log n$)  Heap Sort符合最壞情況O($n log n$)，這4個排序法都符合他們最壞情況的時間複雜度



### 不同排序運行平均時間(500組平均)
| 測試案例 | 輸入參數 $n$ | Insertion Sort | Quick Sort |Merge Sort | Heap Sort |Composite Sorting Function |
|----------|--------------|----------|----------|----------|----------|----------|
| 測試一   | $n = 30$      | 0.000004       | 0.000004       |0.000018     |0.000002        |0.000002      |
| 測試二   | $n = 500$     | 0.000234  | 0.000042   |0.0002      |0.000074  |0.000056       |
| 測試三   | $n = 1000$    | 0.00096  | 0.000102  |0.000492      |0.000156    |0.00011       |
| 測試四   | $n = 2000$    | 0.003678  | 0.000198   |0.001116     |0.000348   |0.000222        |
| 測試五   | $n = 3000$    | 0.011764  | 0.000418   |0.002088     |0.000712   |0.000426         |
| 測試六   | $n = 4000$    | 0.019884  | 0.000478   |0.00261     |0.00092   |0.000666        |
| 測試七   | $n = 5000$    | 0.027722   | 0.000644  |0.003374     |0.001156    |0.00084        |
| 測試八   | $n = 10000$   | 0.13326  | 0.001482  |0.007308     |0.002576    |0.002226        |
| 測試九   | $n = 30000$   | 1.272  | 0.004786   |0.021888     |0.008928   |0.0122722        |
| 測試十   | $n = 50000$   | 3.55914  | 0.00838   |0.038854      |0.016022   |0.026612        |



![平均狀況折線圖](https://cdn.discordapp.com/attachments/930060410823016509/1366437304755028028/B9Nmgo5UxVu2AAAAAElFTkSuQmCC.png?ex=68119a28&is=681048a8&hm=43816d88c0c0ee07828e5f22d17a7d135af9558e00173adbbcfe5cf42fbf05fa&)

根據以上測出來的資料可以看出Insertion Sort符合平均情況(O($n^2$) Quick Sort符合平均情況O($n log n$) Merge Sort符合平均情況O($n log n$)  Heap Sort符合平均情況O($n log n$)，這4個排序法都符合他們平均情況下的時間複雜度

![平均空間折線圖](https://github.com/user-attachments/assets/8e1bda6c-eb51-485f-8732-f9fc28dde12b)



## 申論及開發報告
測試所使用每個排序的最糟情況產生  
插入排序:產生由n、n-1~0由大到小的資料  
快速排序:沿用插入排序的最糟測資，並更改選擇pivot的邏輯，讓pivot每次都選到最大值  
整合排序和堆積排序:利用 permute() 函式實作 Fisher-Yates 洗牌法，確保每一個排列機率相同，隨機產生20組測試資料，並從中選出所需時間最久的  
  
在寫composite sort時，利用了quick sort當作預設的排序方式，因為速度最快。在遞迴深度過高或需排序長度較低時，採用了heap sort和insertion sort，速度相差不多的情況下又能減少記憶體空間占用。




