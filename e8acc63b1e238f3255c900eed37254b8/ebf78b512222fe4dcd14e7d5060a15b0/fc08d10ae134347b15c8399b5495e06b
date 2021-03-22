import 'package:flutter/material.dart';
import 'package:mine/game/game.dart';

class GameStateDisplay extends StatelessWidget {
  final Game game;

  GameStateDisplay(this.game);

  Widget build(BuildContext context) {
    switch (game.state) {
      case GameState.play: return Text('Playing');
      case GameState.loss: return Text('Loss!');
      case GameState.win: return Text('Victory!');
    }
  }
}