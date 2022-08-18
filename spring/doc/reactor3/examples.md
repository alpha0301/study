## 동기에서 비동기로 전환 예제

1. 동기 코드

```java
List<FavoriteDetail> getFavoriteDetails(String userId) {
  try {
    // block
    List<String> favoriteIds = userService.getFavorites(userId);

    if (favoriteIds == null || favoriteIds.isEmpty()) {
      return suggestionService.getSuggestions();
    } else {
      List<FavoriteDetail> details = new LinkedList<>();
      for (String favoriteId : favoriteIds) {
      FavoriteDetail detail = favoriteService.getDetails(favoriteId);
      details.add(detail);
    }
      return details;
    }
  } catch (Exception e) {
    e.printStackTrace();
    throw new RuntimeException(e);
  }
}
```

2. Callbacks을 활용한 비동기 구현

```java
// Thread1
void getFavorites(String userId) {
    // 메소드를 실행한 Thread1은 getFavorites를 호출하고 리턴
    userService.getFavorites(userId, new Callback<List<String>>() {
                // Thread2
                public void onSuccess(List<String> ids) {
                    if (ids == null || ids.isEmpty()) {
                        // Thread2는 getSuggestions를 호출하고 리턴
                        suggestionService.getSuggestions(new Callback<List<FavoriteDetail>>() {
                            // Thread3
                            public void onSuccess(List<FavoriteDetail> details) {
                                // Thread3은 submitOnIoThread 호출
                                // submitOnIoThread 내에서 결과를 전달받은 IoThread가 out.write 수행
                                ResponseUtils.submitOnIoThread(details, new Callback<Void>() {
                                    public void onSuccess(Void result) {
                                        System.out.println("success!");
                                    }

                                    public void onError(Throwable e) {
                                        throw new RuntimeException(e);
                                    }
                                });
                            }

                            public void onError(Throwable e) {
                                throw new RuntimeException(e);
                            }

                        });
                    } else {
                        for (int i = 0; i < ids.size(); i++) {
                            if (i == 5) {
                                break;
                            }
                            
                            favoriteService.getDetails(ids.get(i), new Callback<FavoriteDetail>() {
                                public void onSuccess(FavoriteDetail detail) {
                                    ResponseUtils.submitOnIoThread(detail, new Callback<Void>() {
                                        public void onSuccess(Void result) {
                                            System.out.println("success!");
                                        }

                                        public void onError(Throwable e) {
                                            throw new RuntimeException(e);
                                        }
                                    });
                                }

                                public void onError(Throwable e) {
                                    throw new RuntimeException(e);
                                }
                            });
                        }
                    }
                }

                public void onError(Throwable e) {
                    throw new RuntimeException(e);
                }
            }
    );
}
```
3. Reactor를 활용한 비동기 코드

```java
void getFavoriteDetails(String userId) {
    getFavorites(userId)
        .flatMap((id) -> Flux.just(getFavoriteDetail(id))) // ids 를 받아서 Detail을 어떻게 제공할지 Publisher를 정의
        .switchIfEmpty(Flux.just(suggestDetails())) // 위에서 지정한 Publisher에서 아무것도 제공하지 않았을 때 기본값 세팅
        .take(5) // getDetails가 제공하는 목록 중 5개만 사용
        .publishOn(ioScheduler()) // 이후의 데이터는 io thread가 수행할 수 있도록 설정
        .subscribe(this::writeToClient, this::writeErrorToClient);  // 마지막으로 무엇을 할지 설정하며 trigger the flow
}
```

3-1. 만약 응답 시간을 800ms으로 제어하고 싶다면(서킷브레이커)

```java
void getFavoriteDetails(String userId) {
    getFavorites(userId)
        .timeout(Duration.ofMillis(800))    // getFavorites 가 0.8초 안에 응답이 없으면 중지하고 에러를 발생시킴
        .onErrorResume((e) -> Flux.just(cachedFavoriteIds()))   // 에러를 처리하면서 동시에 다른 데이터로 대체
        .flatMap((id) -> Flux.just(getFavoriteDetail(id)))
        .timeout(Duration.ofMillis(800))    // getDetails 가 0.8초 안에 응답이 없으면 중지하고 에러를 발생시킴
        .onErrorResume((e) -> Flux.just(cachedDetails())) // 에러를 처리하면서 동시에 다른 데이터로 대체
        .switchIfEmpty(Flux.just(suggestDetails()))
        .take(5)
        .publishOn(ioScheduler())
        .subscribe(this::writeToClient, this::writeErrorToClient);
    }
```
