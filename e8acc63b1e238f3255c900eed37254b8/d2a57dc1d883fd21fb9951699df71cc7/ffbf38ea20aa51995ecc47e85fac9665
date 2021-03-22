import 'package:flutter/foundation.dart';

class MineNumSettings extends ChangeNotifier {
  final List<int> mineNums = [1];

  // changes should only be made through this method
  void toggle(int num) {
    if (mineNums.contains(num)) {
      mineNums.remove(num);
    } else {
      mineNums.add(num);
      mineNums.sort();
    }

    notifyListeners();
  }
}