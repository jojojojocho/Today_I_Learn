## JPA 연관관계 매핑과 트랜잭션에 대하여..

#### 오늘 개인 프로젝트에서 게시판의 댓글과 대댓글의 기능을 구현 하면서 문제와 궁금증이 생겼다.

코드로 살펴보자

#### 코멘트 엔티티의 필드
```java

@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "comment_id")
    private Long id;

    @Column(columnDefinition = "longtext")
    private String context;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "board_id")
    private Board board;

    @Enumerated(value = EnumType.STRING)
    private DeleteStatus isDeleted;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Comment parent;

    @OneToMany(mappedBy = "parent", orphanRemoval = true)
    private List<Comment> children = new ArrayList<>();


    private LocalDateTime pubTime;
    private LocalDateTime modTime;

    private Long good;
    private Long bad;

//    @OneToMany(mappedBy = "commentId")
//    private List<ReportComment> reportCommentList=new ArrayList<>();

    private Long reportCount;

```


#### 서비스 단의 코드
```java

    public Long writeComment(CommentDTO commentDTO){

        Board board = boardRepository.findById(commentDTO.getBoardId());
        Member member = memberRepository.findById(commentDTO.getMemberId());
        Comment parent;

        if(commentDTO.getParentId()!=null)
            parent = commentRepository.findById(commentDTO.getParentId());
        else parent=null;

        Comment comment = new Comment(commentDTO.getContext(),member,board, DeleteStatus.FALSE,parent,
                LocalDateTime.now(),LocalDateTime.now(), 0L,0L,0L);

        Long id = commentRepository.save(comment);

        return id;
    }

```

문제 1.

위는 서비스 단의 코드인데 댓글의 내용을 받아 db에 저장하는 코드이다. 

처음에 이 코드를 작성하면서 생각했던 건 코멘트에서 필요한 정보(누가 쓴 댓글인지(Member),  어느 게시글에 속해있는지 (Board))를 em.find로 찾아와 comment 엔티티에 넣어주고,

parentId가 있을 경우와 없을 경우를 나누어 parentId값을 설정해주었을 때 단방향 매핑처럼 실제 db에서 comment의 컬럼에는 memberID, boardId, parentId 이 포함되어 foreign 키 값이 들어가 관리 될 것이라 생각했다. 

그러나 이 과정에서 추가적으로 양방향 매핑을 해주었던 List<Children>에 내가 수동으로 추가해주지 않았음 에도 불구하고, db에 insert 쿼리가 나가는 시점에 저절로 추가가 되는 현상이 발생했다.
  
결과적으로 이 현상이 왜 일어나는지에 답은 찾지 못하였다.
  
JPA를 배울 때에 분명히 양방향 매핑 후 엔티티의 연관관계의 갯수가 늘어날 때마다, 수동적으로 mappedby 를 가진 엔티티에 add메서드를 추가적으로 작성하여, 양방향 매핑을 유지해주어야 한다고 이해하면서 배웠던 것으로 기억한다.
  
그러나 수동으로 추가해주지 않아도 하나의 트랜잭션이 끝나고 hibernate의 insert 쿼리가 나갈때에 저절로 컬렉션에 연관관계에 매핑된 엔티티들이 저절로 추가되는 현상이 발생하니 참으로 당황하지 않을 수 없었다.
  
감히 추측을 해보자면 이 경우에는 연관관계 필드가 모두 하나의 엔티티에 존재하기 때문에 저절로 컬렉션에 추가 되는 특수하는 케이스가 되지 않나 조심스럽게 예상해본다.  

#### 다음은 문제 1이 발생함으로 인해 디버깅을 통해 깨달은 것들을 정리해놓고자 끄적인다.
  
  1. hibernate의 쿼리는 정말 정말 필요하기 전까지는 안 나간다.(lazy 설정해놓을시에 )
  
    예로 컨트롤러 단에서 서비스단의 트랜잭션이 끝났음에도 불구하고 return이 끝나고 난 뒤에 insert 쿼리가 나가는 것을 발견할 수 있었다.
    
    또한 컨트롤러 단에서 2개의 서비스 즉 2개의 트랜잭션이 있을경우 1번째 트랜잭션이 끝난 후에 무작정 insert 쿼리가 나가는 것이 아닌 
  
    2번째 서비스에서 insert 쿼리를 넣어 영속성 컨텍스트를 최신화 해야하는 경우(조회)에 나가는 것을 확인 할 수 있었다. 즉 진짜진짜 쿼리는 맨 마지막에 commit과 동시에 나간다.
  
  2. 위의 진짜진짜 쿼리가 늦게 나가는 경우가 아닌 경우가 있었다. GenerateValue의 전략을 Identity로 설정해놓은 경우였다. identity 전략같은 경우에는 pk값 즉, id값을 db가 결정해주기 때문에
  
    jpa는 db가 값을 정해주기 전까지는 값을 모른다. 즉 db에서 pk값이 결정되고, 그것을 영속성컨택스트에 올려 엔티티로 사용하기 때문에 persist시점에서 바로 insert쿼리가 날아간다.(영속성 컨텍스트에 올려 엔티티를 사용하려면 id 값이 필요하니까.  )
  
    그러나 sequence 전략, table 전략에서는 모아서 한꺼번에 commit을 한다.(위에서 말한 아주아주 마지막 필요하기 전까지 쿼리가 나가지 않음). 즉 버퍼가 가능하다.
    
  
  
  
  
