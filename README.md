## Vanilla JS로 웹 컴포넌트 만들기

[Vanilla Javascript로 웹 컴포넌트 만들기 - 황준일](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Component/#_3-%E1%84%86%E1%85%A9%E1%84%83%E1%85%B2%E1%86%AF%E1%84%92%E1%85%AA)과 **hyunmin**님의 인사이트를 따라해보고 확장해보는 저장소입니다.

### 1. 이벤트 처리
이벤트를 등록하는 과정이 매우 어려웠다. 나는 상황에 맞는 컴포넌트를 그때 렌더링하고 싶고 이제 그때마다 필요한 이벤트가 존재하는데, 렌더링 이후에 이벤트를 등록해야지, 이전에 이벤트를 등록한다면 `undefined`에 이벤트를 등록한다는 에러가 발생하기 때문이다. 그래서 렌더링이 됐다는 것을 캡쳐하는 방법을 생각했지만, 매 컴포넌트마다 이렇게 하기 번거로웠다. 그래서 이벤트를 등록하는 함수를 만들어서 이벤트를 등록하고 싶은 컴포넌트의 `constructor`에서 호출하도록 했다. 이렇게 하면 컴포넌트가 생성될 때마다 이벤트를 등록할 수 있었다.

```javascript
setEvent () {
  this.$target.querySelector('.addBtn').addEventListener('click', () => {
    const { items } = this.$state;
    this.setState({ items: [ ...items, `item${items.length + 1}` ] });
  });

  this.$target.querySelectorAll('.deleteBtn').forEach(deleteBtn =>
    deleteBtn.addEventListener('click', ({ target }) => {
      const items = [ ...this.$state.items ];
      items.splice(Number(target.dataset.index), 1);
      this.setState({ items });
    }))
}
```
하지만 매번 이렇게 이벤트를 새로 등록하는 것은 매우 불편하고 지루한 일이다. 이를 해결할 수 있는 방법이 바로 이벤트 버블링이다.

```javascript
setEvent () {
  // 모든 이벤트를 this.$target에 등록하여 사용하면 된다.
  this.$target.addEventListener('click', ({ target }) => {
    const items = [ ...this.$state.items ];

    if (target.classList.contains('addBtn')) {
      this.setState({ items: [ ...items, `item${items.length + 1}` ] });
    }

    if (target.classList.contains('deleteBtn')) {
      items.splice(Number(target.dataset.index), 1);
      this.setState({ items });
    }

  });
}
```
이 때 이벤트 등록을 `constructor`에서 단 한 번만 실행하도록 한다. 여기서 한 단계 더 추상화하여 깔끔하게 사용할 수 있다.

```javascript
addEvent (eventType, selector, callback) {
  const children = [ ...this.$target.querySelectorAll(selector) ];
  // selector에 명시한 것 보다 더 하위 요소가 선택되는 경우가 있을 땐
  // closest를 이용하여 처리한다.
  const isTarget = (target) => children.includes(target)
    || target.closest(selector);
  this.$target.addEventListener(eventType, event => {
    if (!isTarget(event.target)) return false;
    callback(event);
  })
}
```
