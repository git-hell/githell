import 'package:flutter/material.dart';

import 'package:mine/display/board_display.dart';
import 'package:mine/display/game_state_display.dart';
import 'package:mine/game/game.dart';

abstract class GameDisplay {
  Game get game;
  BoardDisplay get boardDisplay;

  Widget build(BuildContext buildContext) {
    return Column(
      children: [
        Expanded(
          child: FittedBox(
            child: GameStateDisplay(game),
            fit: BoxFit.contain
          ),
          flex: 1
        ),
        Expanded(
          child: Center(
            child: boardDisplay
          ),
          flex: 19
        )
      ]
    );
  }
}