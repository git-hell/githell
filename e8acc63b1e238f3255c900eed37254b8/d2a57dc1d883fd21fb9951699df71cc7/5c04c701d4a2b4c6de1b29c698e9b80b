import 'dart:math';

import 'package:flutter/foundation.dart';

class NeighborSettings extends ChangeNotifier {
  final List<Point<int>> neighbors = [
    Point(0, -1),
    Point(-1, 0),
    Point(1, 0),
    Point(0, 1)
  ];

  void toggle(Point<int> p) {
    if (neighbors.contains(p)) {
      neighbors.remove(p);
    } else {
      neighbors.add(p);
    }

    notifyListeners();
  }
}