# 개요
- BLoC Pattern
- Bussiness Logic Component

# 목적
- UI 코드와 비즈니스 로직 분리
- Flutter 상태 관리를 위해 구글 개발자가 디자인함
- Flutter 는 상태에 따라 랜더링이 일어나므로 관리가 중요함

# 이해
- Reactive Programming 과 유사함
- 각 UI 객체들은 BLoC 객체를 구독함

## BLoC -> UI
- BLoC 객체의 상태가 변하면 구독중인 UI 객체들이 변경됨

## UI -> BLoC
- UI 로부터 Event 를 전달받은 BLoC 객체는 Provider, Repository 를 활용하여 비즈니스 로직을 수행
- 처리 결과를 BLoC -> UI 과정을 통해 상태를 전달

```dart
class BLoC {
  provider: Provider = new Provider();
  stream: Subject = new Subject();
  async sink() {
    const data = await Provider.getCounterModel();
    const result = await this.BussinessLogic(data);
    this.stream.next(result);
  }
  
  private async BussinessLogic() {
    // ...
  }
}

class UI {
  bloc = new BLoC();
  constructor() {
    this.bloc.stream.subscribe((data) => {
      this.render(data);
    });
  }
  render(data) {
    return (
      <div onClick={this.bloc.sink}>
        {data}
      </div>
    )
  }
}
```

