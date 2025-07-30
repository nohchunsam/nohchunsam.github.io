챗지피티한테 리스트 아이템을 옆으로 밀어 삭제하는 위젯과 코드를 짜달라고 했다. 확인해보니 AnimatedList와 Dismissible을 함께 사용했다. 굳이 AnimatedList를 사용할 필요가 있나? 해서 확인해 봤는데, 둘 다 항목에 애니메이션을 적용한다는 건 같지만 AnimatedList가 좀 더 커스텀이 가능했다. 이렇게 알아보는 김에 두 위젯의 차이를 정리하려고 한다.

## AnimatedList

https://youtu.be/ZtfItHwFlZ8?si=UmW1WhNPKblz0-OS

- 리스트 아이템을 애니메이팅한다.
- 기존의 ListVew와 비슷하나, itemBuilder에 animation 파라미터가 있다. 해당 파라미터는 주로 아이템을 애니메이팅하는데 사용된다.
- AnimatedListState를 통해 아이템을 추가하거나 삭제하는 데 애니메이션을 넣을 수 있다.
  - AnimatedListState는 AnimateList용 상태다.
  - AnimatedListState는 insertItem()이나 removeItem() 메서드를 제공한다. 이 메서드를 호출하며 항목을 추가/제거할 때, AnimatedList는 해당 작업에 대한 에니메이션을 실행한다.
  - insertItem을 호출할 때는 AnimatedList.itemBuilder의 애니메이션을, removeItem은 해당 애니메이션의 역방형으로 실행된다.
  - 아래 두가지 방법으로 호출할 수 있다.

```Dart
AniamtedList.of(context).insertItem(index);
AnimatedList.of(context).removerItem(
    index,
    (context, animation) => ...
);

final _myListKey = GlobalKey<AnimatedListState>();
```

### 예시 코드
좀 더 자세히 살펴보자면... 공식 문서에 있는 코드를 파해쳐 보자. 아래의 코드는 이렇게 동작한다.

- ListModel 클래스는 내부 리스트와 AnimatedList 위젯의 상태를 동기화하는 역할을 한다.
- ListModel에서 removeAt()이 호출되면 > _list 항목 제거 > _animatedList.removeItem() 호출 > removedItemBuilder이 호출 > buildRemovedItem이 호출 > 애니메이션이 재생된다.


```Dart
import 'package:flutter/material.dart';

/// AnimatedList 샘플 코드

void main() {
  runApp(const AnimatedListSample());
}

class AnimatedListSample extends StatefulWidget {
  const AnimatedListSample({super.key});

  @override
  State<AnimatedListSample> createState() => _AnimatedListSampleState();
}

class _AnimatedListSampleState extends State<AnimatedListSample> {
  final GlobalKey<AnimatedListState> _listKey = GlobalKey<AnimatedListState>();
  late ListModel<int> _list;
  int? _selectedItem;
  late int  _nextItem; // 사용자가 '+' 버튼을 누르면 다음 아이템이 삽입된다.

  @override
  void initState() {
    super.initState();
    _list = ListModel<int>(
      listKey: _listKey,
      initialItems: <int>[0, 1, 2],
      removedItemBuilder: _buildRemovedItem,
    );
    _nextItem = 3;
  }

  // 제거되지 않은 리스트 아이템을 생성하는데 사용.
  Widget _buildItem(
    BuildContext context,
    int index,
    Animation<double> animation,
  ) {
    return CardItem(
      animation: animation,
      item: _list[index],
      selected: _selectedItem == _list[index],
      onTap: () {
        setState(() {
          _selectedItem = _selectedItem == _list[index] ? null : _list[index];
        });
      },
    );
  }

  /// 제거된 리스트 아이템을 생성하는데 사용되는 빌드 함수.
  ///
  /// 리스트에서 삭제된 후의 아이템을 생성하는데 사용됩니다.
  /// 이 함수는 애니메이션이 완료될 때까지 볼 수 있기 때문에 필요합니다.
  /// (이 ListModel에 관해서는 이미 이 단계까지 진행되었지만) 
  /// 이 위젯은 [AnimatedListState.removeItem] 메서드의  
  /// [AnimatedRemovedItemBuilder] 매개변수에 의해 사용됩니다.
  Widget _buildRemovedItem(
    int item,
    BuildContext context,
    Animation<double> animation,
  ) {
    return CardItem(
      animation: animation,
      item: item,
      // 제스처 디텍터 X: 이미 제거된 아이템이 상호작용 가능해서는 안됩니다.
    );
  }

  // 리스트 모델에 다음 아이템을 삽입합니다.
  void _insert() {
    final int index = _selectedItem == null
        ? _list.length
        : _list.indexOf(_selectedItem!);
    _list.insert(index, _nextItem);
    _nextItem++;
  }

  // 리스트 모델에서 선택된 아이템을 제거합니다.
  void _remove() {
    if (_selectedItem != null) {
      _list.removeAt(_list.indexOf(_selectedItem!));
      setState(() {
        _selectedItem = null;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('AnimatedList'),
          actions: <Widget>[
            IconButton(
              icon: const Icon(Icons.add_circle),
              onPressed: _insert,
              tooltip: 'insert a new item',
            ),
            IconButton(
              icon: const Icon(Icons.remove_circle),
              onPressed: _remove,
              tooltip: 'remove the selected item',
            ),
          ],
        ),
        body: Padding(
          padding: const EdgeInsets.all(16.0),
          child: AnimatedList(
            key: _listKey,
            initialItemCount: _list.length,
            itemBuilder: _buildItem,
          ),
        ),
      ),
    );
  }
}

typedef RemovedItemBuilder<T> =
    Widget Function(T item, BuildContext context, Animation<double> animation);

/// Dart [List]를 [AnimatedList]와 동기화합니다.
///
/// [insert] 및 [removeAt] 메서드는 내부 목록과
/// [listKey]에 속하는 애니메이션 목록 모두에 적용됩니다.
///
/// 이 클래스는 샘플 앱에서 필요한 만큼만 Dart List API를 노출합니다.
/// 추가 목록 메서드는 쉽게 추가할 수 있지만, 목록을 변경하는 메서드는
/// [AnimatedListState.insertItem] 및 [AnimatedListState.removeItem]을 통해
/// 애니메이션 목록에도 동일한 변경을 적용해야 합니다.
class ListModel<E> {
  ListModel({
    required this.listKey,
    required this.removedItemBuilder,
    Iterable<E>? initialItems,
  }) : _items = List<E>.from(initialItems ?? <E>[]);

  final GlobalKey<AnimatedListState> listKey;
  final RemovedItemBuilder<E> removedItemBuilder;
  final List<E> _items;

  AnimatedListState? get _animatedList => listKey.currentState;

  void insert(int index, E item) {
    _items.insert(index, item);
    _animatedList!.insertItem(index);
  }

  E removeAt(int index) {
    final E removedItem = _items.removeAt(index);
    if (removedItem != null) {
      _animatedList!.removeItem(index, (
        BuildContext context,
        Animation<double> animation,
      ) {
        return removedItemBuilder(removedItem, context, animation);
      });
    }
    return removedItem;
  }

  int get length => _items.length;

  E operator [](int index) => _items[index];

  int indexOf(E item) => _items.indexOf(item);
}

/// 카드의 색상이 항목의 값에 따라 결정되는 카드에 정수 항목을 '항목 N'으로 표시합니다.
///
///
/// [selected]가 true인 경우 텍스트는 밝은 녹색으로 표시됩니다. 이 위젯의 높이는
/// [animation] 매개변수에 따라 결정되며, 애니메이션이 0.0에서 1.0으로 변동될 때
/// 0에서 128까지 변동됩니다.
class CardItem extends StatelessWidget {
  const CardItem({
    super.key,
    this.onTap,
    this.selected = false,
    required this.animation,
    required this.item,
  }) : assert(item >= 0);

  final Animation<double> animation;
  final VoidCallback? onTap;
  final int item;
  final bool selected;

  @override
  Widget build(BuildContext context) {
    TextStyle textStyle = Theme.of(context).textTheme.headlineMedium!;
    if (selected) {
      textStyle = textStyle.copyWith(color: Colors.lightGreenAccent[400]);
    }
    return Padding(
      padding: const EdgeInsets.all(2.0),
      child: SizeTransition(
        sizeFactor: animation,
        child: GestureDetector(
          behavior: HitTestBehavior.opaque,
          onTap: onTap,
          child: SizedBox(
            height: 80.0,
            child: Card(
              color: Colors.primaries[item % Colors.primaries.length],
              child: Center(child: Text('Item $item', style: textStyle)),
            ),
          ),
        ),
      ),
    );
  }
}
```

이런 식으로 animatedList가 작동한다. AnimatedList는 아이템 삭제 시 애니메이션을 실행한다. 

insertItem에 별도의 애니메이션을 추가하지 않는 이유는, 아이템이 추가될 때마다 자동으로 애니메이션을 전당해주기 때문이다. 그 덕에 insertItem 파라미터에는 builder 함수가 없다.

> +) Globalkey: 전체 앱에서 고유한 키이다.

## Dismissible

https://youtu.be/iEMgjrfuc58?si=9GH8WI2j3CNAcPqS

- 특정 방향을 드래그해 닫을 수 있는 위젯이다. 즉, 위의 AnimatedList와 달리 애니메이션이 이미 지정돼 있다.
- 드래그 방향, 속도 등을 설정할 수 있다.
- 드래그할 때 보이는 UI(예, 삭제 아이콘 등)를 최대 2개까지 설정할 수 있다.
- 드래그할 때, 삭제든 함수를 지정한다.

...즉, 정말 위젯을 드래그해서 실행하는 위젯 그 이상도 이하도 아니다!!

-----

나는 오직 밀어서 삭제 기능만 필요하기에 Dismissible만 사용했다.
물론 아이템 추가 애니메이션을 넣어 둘 다 사용해도 되지만... 아이템 추가에 별도의 애니메이션을 넣을 생각은 없기에 AnimatedList를 제거했다.

요약: 단순히 리스트 아이템을 밀어서 삭제하고 싶으면 Dismissible 하나로 충분하다!