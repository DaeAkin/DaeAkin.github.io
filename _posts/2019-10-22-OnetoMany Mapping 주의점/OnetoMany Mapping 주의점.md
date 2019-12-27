---
layout: post
title: OnetoMany Mapping 주의점
categories: [JPA]
comments: true

---
# @OnetoMany Mapping 주의점

회사에서 이미 만들어진 ERD의 테스트코드를 작성하고 있었는데, 데이터를 하나 추가할 때 마다

delete 동작이 한번, insert 동작이 N번이 되는 현상이 있었습니다.

이 현상을 테스트 하기 위해 2개의 테이블을 만들어서 테스트해보겠습니다.



## 1. 테스트코드 작성하기

### 1-1.Board

```java
@Entity
@Data
@NoArgsConstructor
public class Board {
    @Id
  	@GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "BOARD_ID")
    private Long id;

    private String title;

  	//단방향 
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinTable(name="BOARD_IMAGE",
            joinColumns = @JoinColumn(name = "BOARD_ID"),
            inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
     List<Image> imageList = new ArrayList<>();

    public Board(String title) {
        this.title = title;
    }
}
```

> ### NOTE
>
> 일대다에서 기본 Join 방법은 JoinTable 입니다. 
>
> 만약 JoinColumn을 사용할 계획이시라면 @JoinColumn 어노테이션을 작성하여 명시해줘야합니다.
>
> 위에 나와있는 Board 코드는 @JoinTable을 생략해도 됩니다.

### 1-2. Image

```java
@Entity
@Data
@NoArgsConstructor
public class Image {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "IMAGE_ID")
    private Long id;

    private String imageUrl;

    public Image(String imageUrl) {
        this.imageUrl = imageUrl;
    }
}
```

JPA가 그려준 ERD를 보면 다음과 같습니다.

### 1-3. ERD

![](https://github.com/DaeAkin/DaeAkin.github.io/blob/master/_posts/2019-10-22-OnetoMany%20Mapping%20%EC%A3%BC%EC%9D%98%EC%A0%90/img/erd.png?raw=true)

@JoinTable 이용했기 때문에 board_image 테이블이 하나 더 만들어졌습니다.

그리고 board -> image 는 1:N 관계 입니다.



### 1-4.Test 코드

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class OnetoManyErrorTests {

    @Autowired
    BoardRepository boardRepository;

    @Test
    public void test() {
        System.out.println("save 1 Board and 2 Image Start");
        Board board = new Board("test Board");
        board.imageList.add(new Image("file://aaa.png"));
        board.imageList.add(new Image("file://bbb.png"));
        board = boardRepository.save(board);
        System.out.println("save 1 Board and 2 Image End");

        System.out.println("Add one Image");
        board.imageList.add(new Image("file://ccc.png"));
        boardRepository.save(board);
        System.out.println("Add one Image End");

    }
}
```

[코드 보러가기](https://github.com/DaeAkin/Spring-Jpa/blob/oneToManyBefore/src/test/java/com/donghyeon/springJpa/onetomanyerror/OnetoManyErrorTests.java)

### 1-5. Console 결과

```
save 1 Board and 2 Image Start
Hibernate: insert into board (title) values (?)
Hibernate: insert into image (image_url) values (?)
Hibernate: insert into image (image_url) values (?)
Hibernate: insert into board_image (board_id, child_id) values (?, ?)
Hibernate: insert into board_image (board_id, child_id) values (?, ?)
save 1 Board and 2 Image End

Add one Image
Hibernate: select board0_.board_id as board_id1_0_1_, board0_.title as title2_0_1_, imagelist1_.board_id as board_id1_1_3_, image2_.image_id as child_id2_1_3_, image2_.image_id as image_id1_2_0_, image2_.image_url as image_ur2_2_0_ from board board0_ left outer join board_image imagelist1_ on board0_.board_id=imagelist1_.board_id left outer join image image2_ on imagelist1_.child_id=image2_.image_id where board0_.board_id=?
Hibernate: insert into image (image_url) values (?)
//delete 한번과 list.size()만큼 insert를 한다.
Hibernate: delete from board_image where board_id=?
Hibernate: insert into board_image (board_id, child_id) values (?, ?)
Hibernate: insert into board_image (board_id, child_id) values (?, ?)
Hibernate: insert into board_image (board_id, child_id) values (?, ?)
Add one Image End
```

1개의 board에 2개의 image를 저장한 후 추가로 새로운 1개의 image를 추가했더니, 

해당되는 board.id의 board_image 테이블의 데이터들을 전부 삭제하고,

list.size()만큼 3개의 데이터를 insert를 했습니다.

만약 99개의 list가 있었다면, 데이터를 추가를 하게되면 총 100개의 insert 쿼리문이 날라가게되어 성능 저하를 일으킬 수 있습니다.



## 2. 이유

이렇게 collection으로 맵핑을 이용해 joinTable을 이용하게 되면 하이버네이트는 이전 collection을 전부 삭제합니다.

그 다음 joinTable에 새로운 collection들을 집어 넣기 때문에 N번 insert 이슈가 나타나게 됩니다.

하하이버네이트는 collection 안에있는 요소들이 몇개나 변화됐는지 분석을 하지 않기 때문에 생기는 일입니다.



## 3. 해결방안 

### 3-1. 인덱스화

해결방안으로는 id값을 주어서 리스트들을 index화 해야합니다. 그래야 하이버네이트가 변화를 감지할 수 있습니다.

### 3-2. @ManytoOne 사용하기

2개의 Entity를 새로운 Entity가 관리하는 방법입니다.



## 4. 해결해보기

### 4-1. Board

```java
@Entity
@Data
@NoArgsConstructor
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "BOARD_ID")
    private Long id;

    private String title;


    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true,mappedBy = "board")
    List<BoardAndImage> boardAndImageList = new ArrayList<>();

    public Board(String title) {
        this.title = title;
    }
}
```



### 4-2. BoardAndImage

```java
@Entity
@Data
@NoArgsConstructor
public class BoardAndImage {

    @Id @Column(name = "BOARDANDIMAGE_ID")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "BOARD_ID")
    private Board board;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name="IMAGE_ID")
    private Image image;

    public BoardAndImage(Board board, Image image) {
        this.board = board;
        this.image = image;
    }
}
```



### 4-3 Image 

image Entity는 변화 없습니다.



## 5. 검증

### 5-1 Test 코드

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class OnetoManyErrorTests {

    @Autowired
    BoardRepository boardRepository;

    @Test
    public void test() {
        System.out.println("save 1 Board and 2 Image Start");
        Board board = new Board("test Board");
        board.boardAndImageList.add(new BoardAndImage(board,new Image("file://aaa.png")));
        board.boardAndImageList.add(new BoardAndImage(board,new Image("file://bbb.png")));
        board = boardRepository.save(board);
        System.out.println("save 1 Board and 2 Image End");

        System.out.println("Add one Image");
        board.boardAndImageList.add(new BoardAndImage(board,new Image("file://ccc.png")));
        boardRepository.save(board);
        System.out.println("Add one Image End");

    }
}
```

[변경 후 코드](https://github.com/DaeAkin/Spring-Jpa/blob/oneToManyAfter/src/test/java/com/donghyeon/springJpa/onetomanyerror/OnetoManyErrorTests.java)



### 5-2 Console 결과

```
save 1 Board and 2 Image Start
Hibernate: insert into board (title) values (?)
Hibernate: insert into image (image_url) values (?)
Hibernate: insert into board_and_image (board_id, image_id) values (?, ?)
Hibernate: insert into image (image_url) values (?)
Hibernate: insert into board_and_image (board_id, image_id) values (?, ?)
save 1 Board and 2 Image End

Add one Image
Hibernate: select board0_.board_id as board_id1_0_2_, board0_.title as title2_0_2_, boardandim1_.board_id as board_id2_1_4_, boardandim1_.boardandimage_id as boardand1_1_4_, boardandim1_.boardandimage_id as boardand1_1_0_, boardandim1_.board_id as board_id2_1_0_, boardandim1_.image_id as image_id3_1_0_, image2_.image_id as image_id1_2_1_, image2_.image_url as image_ur2_2_1_ from board board0_ left outer join board_and_image boardandim1_ on board0_.board_id=boardandim1_.board_id left outer join image image2_ on boardandim1_.image_id=image2_.image_id where board0_.board_id=?

//2번의 insert문만 있음.
Hibernate: insert into image (image_url) values (?)
Hibernate: insert into board_and_image (board_id, image_id) values (?, ?)
Add one Image End
```



## 6. 참고자료

https://stackoverflow.com/questions/24580527/hibernate-recreates-join-table-when-adding-to-list
