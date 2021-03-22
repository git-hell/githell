import 'dart:math';

import 'package:flutter/material.dart';

import 'package:mine/board/multi_mine_board.dart';
import 'package:mine/display/board_display.dart';
import 'package:mine/game/multi_mine_game.dart';

class DoubleDisplay extends StatelessWidget {
  final Widget icon;
  final Widget numWidget;

  DoubleDisplay(this.icon, this.numWidget);

  Widget build(BuildContext context) {
    Widget wrap(Widget w) => Expanded(child: FittedBox(child: w, fit: BoxFit.contain));
    return Column(
      children: [
        Expanded(child: Row(
          children: [wrap(icon), wrap(Container())],
          crossAxisAlignment: CrossAxisAlignment.stretch
        )),
        Expanded(child: Row(
          children: [wrap(Container()), wrap(numWidget)],
          crossAxisAlignment: CrossAxisAlignment.stretch
        )),
      ],
    );
  }
}

class MultiMineBoardDisplay<T extends MultiMineGame> extends BoardDisplay<T> {
  final T game;

  MultiMineBoardDisplay(this.game);

  Widget getDisplay(Point p) {
    MultiMineSquare sq = game.board.boardMap[p]!;

    if (!sq.isRevealed) {
      if (sq.isFlagged) {
        return DoubleDisplay(flag, Text(sq.flag.toString()));
      } else {
        return BoardDisplay.fitWrap(emptySquare);
      }
    }

    if (sq.isMine) {
      return DoubleDisplay(mine, Text(sq.mineNum.toString()));
    } else {
      return BoardDisplay.fitWrap(numWidget(game.board.neighborMines(p)));
    }
  }

  State<MultiMineBoardDisplay> createState() => _MultiMineBoardDisplayState();
}

class _MultiMineBoardDisplayState extends BoardDisplayState<MultiMineBoardDisplay> {}