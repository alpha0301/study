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
List<FavoriteDetail> getFavoriteDetails(String userId, OutputStream out) {
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
