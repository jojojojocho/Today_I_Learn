```java

public Board findByMemberIdAndPubTime(BoardDTO boardDTO){
        Member member = em.find(Member.class, boardDTO.getMemberId());
        List<Board> result= em.createQuery("select b from Board b where b.member = :member and b.pubTime = :pubTime",
                        Board.class)
                .setParameter("member", member)
                .setParameter("pubTime", boardDTO.getPubTime())
                .getResultList();
}    
public Long delete(Member member){
        Member member1 = em.find(Member.class, member.getId());
        List<Board> result = em.createQuery("select b from Board b where b.member = :member ", Board.class)
                .setParameter("member", member1)
                .getResultList();
}

```

## 기억하기 !!
 - jpa 쿼리를 사용할 때에는 항상 엔티티.xxx(ex b.xxx) 를 엔티티의 기준에서 생각해야 하고, 파라미터를 넣어줄 때에도 엔티티의 자료구조 기준으로 값을 넣어 주어야 한다.
