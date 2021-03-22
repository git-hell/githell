import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'package:mine/app/difficulty_settings.dart';
import 'package:mine/game/default_difficulties.dart';
import 'package:mine/game/difficulty.dart';

class DifficultySelection extends StatelessWidget {
  static final difficulties = [
    DefaultDifficulties.beginner,
    DefaultDifficulties.intermediate,
    DefaultDifficulties.expert
  ];

  Widget build(BuildContext context) {
    return Consumer<DifficultySettings>(
      builder: (_, settings, __) => ListView(
        children: difficulties.map(
          (d) => RadioListTile<Difficulty>(
            title: Text(d.toString()),
            value: d,
            groupValue: settings.difficulty,
            onChanged: (Difficulty? d) { settings.difficulty = d!; }
          )
        ).toList()
      )
    );
  }
}