## - 키워드 정리

#### - 지연로딩과 즉시로딩의 차이

- **즉시 로딩**

    - 연관된 엔티티를 즉시, 같이 한 번에 조회한다.

    - 객체를 조회하는 순간 연관 객체도 함께 로딩된다.

    - 장점

        - 조회 성능이 일관된다.

        - N + 1 문제가 발생하지 않는다.

    - 단점

        - 불필요한 JOIN이 발생한다.

        - 쿼리 복잡도가 증가한다.

        - 양방향 연관관계에서 무한 로딩이 발생할 수 있다.

- **지연 로딩**

    - 연관된 엔티티를 **실제로 사용할 때까지 조회를 미룬다.**

    - 연관된 엔티티는 프록시 객체로 채워진다.

    - 장점

        - 불필요한 JOIN 없이 필요한 데이터만 가져오므로 성능이 최적화된다.

        - 유연하게 제어할 수 있다.

    - 단점

        - N + 1 문제가 발생할 수 있다.

        - 트랜잭션 밖에서 지연로딩된 객체에 접근하면 예외가 발생할 수 있다.

-> **실무에서는 항상 LAZY로 설정하고, 필요한 곳만 Fetch Join을 적용한다.**

#### - Fetch Join

- JPA에서 지연로딩 설정된 연관관계를 한 번의 쿼리로 같이 가져오기 위해 사용하는 방법

- 일반 Join은 영속성 컨텍스트에 반영되지 않지만, **Fetch Join은 조인한 결과를 영속성 컨텍스트에 등록까지 해준다.**

- Fetch Join을 사용하면 지연 로딩으로 했을 때의 장점을 가져가면서, N + 1 문제를 해결할 수 있다.

- 주의점

    - 중복 데이터 문제가 발생한다. (1:N 관계면 N개의 데이터 생성)

    - 페이징이 불가능하다.

    - 2개 이상 Fetch Join시 오류 혹은 성능 저하가 발생한다.

#### - @EntityGraph

- JPA에서 지연로딩으로 설정된 연관 엔티티를 명시적으로 즉시 로딩하도록 지시하는 어노테이션

- Fetch Join과 똑같은 쿼리가 나간다.

- 사용 예시

  ``` java
  @EntityGraph(attributePaths = {"team"})
  List<Member> findAll();  // Member와 team을 한방에 로딩
  ```

- 주의점

    - Fetch Join과 마찬가지로, 페이징이 불가능하다.

#### - JPQL

- Java 객체(Entity)를 기준으로 작성하는 쿼리 언어

- Java Persistence Query Language

- 테이블과 컬럼이 아닌, 엔티티와 필드를 대상으로 쿼리를 작성한다.

- 사용 예시

  ``` java
  @Query("SELECT m FROM Member m JOIN m.team t WHERE t.name = :teamName")
  List<Member> findByTeamName(@Param("teamName") String teamName);
  ```

#### - QueryDSL

- 자바 코드로 SQL과 유사한 쿼리를 타입 안전하게 작성하게 해줄 수 있는 DSL(Domain Specific Language)

- 사용 예시

  ``` java
  query.select(
     Projections.constructor(MemberDto.class,
        m.username,
        m.age))
     .from(m)
     .fetch();
  ```

#### - JPQL vs QueryDSL

- **JPQL**

    - 장점

        - 코드가 간결하다.

        - JPA 공식 스펙에 포함되어 있다.

        - 별도의 라이브러리를 설치하지 않아도 된다.

        - 빠르게 익힐 수 있다.

    - 단점

        - 오타, 필드명 오류 등이 **컴파일 타임이 아닌 런타임 에러로 발생한다.**

        - **동적 쿼리를 작성하기가 힘들다.**

        - IDE 자동완성 지원이 잘 되지 않는다.

        - 엔티티 구조가 바뀌면 문자열을 다 찾아서 수정해야 한다.

- **QueryDSL**

    - 장점

        - **컴파일 타임에 문법, 필드명, 타입 체크가 가능하다.**

        - IDE 자동완성이 지원된다.

        - 동적 쿼리를 쉽게 작성할 수 있다.

        - 페이징/서브쿼리도 지원한다.

    - 단점

        - 별도의 라이브러리 설치 및 설정이 필요하다.

        - 문법/구조를 익혀야 한다.

        - 간단한 쿼리를 작성할 때도 길이가 조금 길어진다.

-> **간단한 조회 쿼리는 JPQL로**, **복잡한 동작 쿼리, 타입 안정성이 중요하면 QueryDSL**을 사용하자.

## - 시니어 미션

이미 한 차례 QueryDSL로 리팩토링을 해서 간단하게만 남기겠습니다 ㅠㅠ

저는 트리 형식의 폴더 구조를 구현해야 하는 프로젝트를 진행했고, 복잡한 쿼리가 익숙하지 않고, JpaRepository 인터페이스 구현체에 익숙했던 상태였습니다.

그 당시에는 다음과 같이 서비스 로직에서 불필요하게 코드를 작성했던 기억이 있습니다.

```java
@Override
    public FolderResponseDto findFolder(Long userId, Long folderId) {

        Optional<Folder> optionalFolder = folderRepository.findById(folderId);

        if (optionalFolder.isPresent() && optionalFolder.get().getUser().getId().equals(userId)) {

            Folder folder = optionalFolder.get();

            FolderThumbnailResponseDto parentFolder = null;
            if (folder.getParentFolder() != null) {
                parentFolder = FolderThumbnailResponseDto.builder()
                        .folderId(folder.getParentFolder().getId())
                        .folderName(folder.getParentFolder().getName())
                        .build();
            }

            List<FolderThumbnailResponseDto> subFolders = null;

            if (folder.getSubFolders() != null && !folder.getSubFolders().isEmpty()) {
                subFolders = folder.getSubFolders().stream().map(subFolder -> FolderThumbnailResponseDto.builder()
                        .folderId(subFolder.getId())
                        .folderName(subFolder.getName())
                        .build()).toList();
            }

            List<ProblemResponseDto> problems = problemService.findAllProblemsByFolderId(folderId);

            return FolderResponseDto.builder()
                    .folderId(folder.getId())
                    .folderName(folder.getName())
                    .parentFolder(parentFolder)
                    .subFolders(subFolders)
                    .problems(problems)
                    .updateAt(folder.getUpdatedAt())
                    .createdAt(folder.getCreatedAt())
                    .build();
        }

        throw new FolderNotFoundException("공책을 찾을 수 없습니다!, folderId: " + folderId);
    }
```

하지만 QueryDSL을 사용해 N + 1 문제를 해결하면서 복잡한 쿼리 없이 조회 쿼리를 완성했습니다.

```java
@Override
    public Optional<Folder> findFolderWithDetailsByFolderId(Long folderId) {
        Folder folder = queryFactory.selectFrom(QFolder.folder)
                .leftJoin(QFolder.folder.subFolderList, new QFolder("subFolder")).fetchJoin()
                .leftJoin(QFolder.folder.parentFolder, new QFolder("parentFolder")).fetchJoin()
                .where(QFolder.folder.id.eq(folderId))
                .fetchOne();

        return Optional.ofNullable(folder);
    }
```

- **@Transactional(readOnly = true) 적용 여부에 따른 실행 속도 차이 테스트**

    - Transactional(readOnly=true)란?
        - 트랜잭션을 읽기 전용으로 설정한다.

        - 트랜잭션 범위 안에서 생성, 수정, 삭제가 발생해도 flush 시점에 변경 내용을 DB에 반영하지 않는다.

        - 엔티티 스냅샷을 유지하지 않아 더티체킹 비용이 사라진다.

        - 메서드가 조회만 담당한다는 것을 명확히 표현해준다.

    - 조사 결과 1000건 이하에서는 5ms 미만의 차이가 발생하지만, 10000건 이상의 대규모 데이터에서는 20%의 개선까지 관찰된다고 합니다.

- **@BatchSize(size = 100)설정 후, 쿼리 실행 횟수 변화 확인**

    - BatchSize는 왜 쓰는걸까?
        - @BatchSize(size = X) 를 설정하면,
          Hibernate가 최대 X개씩 묶어서 한 번에 SELECT ... WHERE id IN (...) 형태로 가져온다.        
          → 1 + N 개가 1 + [N/X]개로 줄어든다.

        - ex) A를 1번 조회할 때 B를 1000개 지연로딩으로 가져온다고 치면, Batchsize=100을 적용했을 때,  A 1번, B 10번의 쿼리만으로 가져올 수 있다. (1000/100 = 10)

            - 만약 B에 250개의 데이터가 있다고 치면 250/100 = 3개의 쿼리만으로 가져올 수 있다.

    - Fetch join과의 차이가 뭘까?

        - Fetch join은 언제나 1번의 쿼리가 나가지만, **조인된 행이 곱해져 결과가 중복된다.**

        - Batchsize는 쿼리 결과에 중복이 없어 각 배치별로 고유한 연관 데이터만 조회된다.

    - 하지만, **BatchSize가 너무 크면 SQL 문자열 길이가 길어지고, 네트워크로 전송되는 패킷 크기가 커져 오히려 응답 시간이 늘어날 수도 있기 때문에**, 적당한 BatchSize를 설정하는 것이 중요하다.
  