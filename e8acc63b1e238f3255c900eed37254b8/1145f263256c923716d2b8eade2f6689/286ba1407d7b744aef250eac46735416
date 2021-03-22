import 'dart:math';

import 'package:mine/board/board.dart';

class MultiMineSquare implements Square {
  int mineNum;
  bool get isMine => mineNum > 0;

  bool isRevealed = false;

  // 0 if not flagged
  int flag = 0;
  bool get isFlagged => flag > 0;

  MultiMineSquare(this.mineNum);
}

class MultiMineBoard extends Board {
  final int width;
  final int height;
  final int mines;

  final Map<Point, MultiMineSquare> boardMap = {};

  MultiMineBoard(this.width, this.height, this.mines) {
    for (int x = 0; x < width; x++) {
      for (int y = 0; y < height; y++) {
        boardMap[Point(x, y)] = MultiMineSquare(0);
      }
    }
  }

  int neighborMines(Point point) {
    int sum = 0;
    for (var np in getNeighbors(point)) {
      sum += boardMap[np]!.mineNum;
    }
    return sum;
  }
}