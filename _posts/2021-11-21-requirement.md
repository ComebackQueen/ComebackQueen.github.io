---
layout: single
title:  "서블릿, JSP, MVC 패턴"
toc: true
toc_sticky: true
categories:
  - Spring_MVC_ver1
---

# 회원 관리 웹 애플리케이션 요구사항



- **회원 정보**
  
  - 이름 : username
  
  - 나이 : age
  
    
  
- **기능 요구사항**
  
  - 회원 저장
  - 회원 목록 조회

---



- **회원 도메인 모델**

```java
@Getter @Setter
public class Member {

    private Long id; // Member를 회원 저장소에 저장하면 회원 저장소가 할당
    private String username;
    private int age;

    public Member() {
    }

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

---



- **회원 저장소**

```java
public class MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    private static final MemberRepository instance = new MemberRepository(); // 싱글톤 사용 (한번의 new로 인스턴스를 사용 -> 메모리 낭비 방지)

    public static MemberRepository getInstance() {
        return instance;
    }

    private MemberRepository() {

    }

    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```

