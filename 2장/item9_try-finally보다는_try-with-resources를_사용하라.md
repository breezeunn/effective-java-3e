# 핵심 정리
**꼭 회수해야 하는 자원을 다룰 때는 try-finally 가 아닌 try-with-resource 를 사용하자.**
* 코드가 짧고, 분명해진다.
* 만들어지는 예외 정보도 유용하다.
* 정확하고 쉽게 자원을 회수할 수 있다.   


### 아래 두 try-finally 의 미묘한 결점
try 블록과 finally 블록 모두에서 예외가 발생하는 경우(예: 기기에 물리적인 문제가 생겨서, readLine() 이 예외를 던지고, 같은 이유도 close() 도 실패하다면?), 첫번째 예외에 관한 정보는 스택 추적 내역에 남지 않게 된다.


### 9-1. try-finally
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

### 9-3. 9-1 을 try-with-resources 로 변경
파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하도록 변경
```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```


### 9-2. 자원이 둘 이상이라면, 더 지저분해지는 try-finally
```java
static void copy(String src String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = inread(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

### 9-4. 9-2 를 try-with-resources 로 개선!
```java
static void copy(String src, String dst) throws IOException {
    try (IputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
             byte[] buf = new byte[BUFFER_SIZE];
             int n;
             while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
         }    
}
```

### try-with-resource (자바 7~)
* 이 구조를 사용하기 위해서는 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다. (void 를 반환하는 close 메서드 하나만 있는 인터페이스)   
* 실제 BufferedReader 의 api doc 을 보면 AutoCloseable 를 구현함을 알 수 있다.

```
java.io  
Class BufferedReader  
java.lang.Object  
java.io.Reader  
java.io.BufferedReader  
All Implemented Interfaces:
Closeable, AutoCloseable, Readable
```
