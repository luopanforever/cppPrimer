## 读取一个字符

### int ch = cin.get();

- 返回值是`int`，从缓冲区读入一个字符

### cin.get(ch);

- 从缓冲区读入一个字符到`ch(char类型)`中

### 读取一些字符

### cin.get(char* s, streamsize n, char delim = '\n');

- 从缓冲区中读入`n-1`个字符，默认遇见换行符停止读取

## cin.get()与cin.getline()的区别

- cin.get()每次读取一整行并把由Enter键生成的**换行符留在输入队列缓冲区**中，然而cin.getline()每次读取一整行并把由Enter键生成的**换行符抛弃**,比如																																																																																																																																																																																																										

## cin.getline(char * cname, int length, char delim = '\n');