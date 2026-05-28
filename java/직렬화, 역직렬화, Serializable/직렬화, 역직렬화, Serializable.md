# 직렬화, 역직렬화, Serializable

## 직렬화

직렬화는 데이터 구조나 객체의 상태를 저장하거나 전송할 수 있는 형태로 변환하는 과정이다.

객체는 메모리 주소를 포함한 상태로 존재하기 때문에, 다른 프로세스나 다른 컴퓨터에 객체 인스턴스 자체를 그대로 전달할 수 없다. 따라서 주소값이 아니라 전송 가능한 바이트 스트림 또는 문자열 포맷 등의 데이터 형태로 변환해야 한다.

- Java Serializable: 객체를 바이트 스트림으로 변환
- JSON 직렬화: 객체를 문자열 기반 포맷으로 변환

## 역직렬화

역직렬화는 직렬화된 데이터를 다시 원래의 객체나 데이터 구조로 복구하는 과정이다.

## 사용 예시

1. 네트워크 통신
   - 클라이언트와 서버 간에 데이터를 전송할 때 사용한다.
   - 송신 측은 객체를 직렬화하고, 수신 측은 데이터를 역직렬화해서 사용한다.
2. 데이터 저장
   - 객체의 상태를 파일 시스템이나 데이터베이스에 저장할 때 사용한다.
   - 저장된 데이터를 다시 읽어 역직렬화하면 기존 객체 상태를 복원할 수 있다.
3. 캐싱
   - 자주 사용하는 데이터를 캐시에 저장할 때 사용할 수 있다.
   - 필요할 때 캐시 데이터를 역직렬화해서 다시 사용한다.

## Java Serializable

`Serializable`은 내부 메서드가 없는 마커 인터페이스이다.
클래스가 `Serializable`을 구현하면 Java의 기본 직렬화 대상이 될 수 있다.

```java
import java.io.Serializable;

class Person implements Serializable {
    private static final long serialVersionUID = 1L;

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

> ![직렬화, 역직렬화, Serializable 1](%EC%A7%81%EB%A0%AC%ED%99%94%2C%20%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94%2C%20Serializable%201.png)
> 
> serialVersionUID는 직렬화된 클래스의 버전 식별자이다. 역직렬화할 때 저장된 클래스의 serialVersionUID와 현재 클래스의 serialVersionUID가 다르면 InvalidClassException이 발생한다.


직렬화는 `ObjectOutputStream`의 `writeObject()`로 수행한다.

```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;

public static void main(String[] args) throws Exception {
    Person person = new Person("John Doe", 30);

    try (FileOutputStream fileOut = new FileOutputStream("person.ser");
         ObjectOutputStream out = new ObjectOutputStream(fileOut)) {
        out.writeObject(person);
    }
}
```

역직렬화는 `ObjectInputStream`의 `readObject()`로 수행한다.

```java
import java.io.FileInputStream;
import java.io.ObjectInputStream;

public static void main(String[] args) throws Exception {
    Person person;

    try (FileInputStream fileIn = new FileInputStream("person.ser");
         ObjectInputStream in = new ObjectInputStream(fileIn)) {
        person = (Person) in.readObject();
    }
}
```
