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
