import 'package:flutter/material.dart';
import 'package:mine/app/neighbor_settings.dart';
import 'package:mine/app/settings_confirmation.dart';
import 'package:provider/provider.dart';

import 'package:mine/app/difficulty_settings.dart';
import 'package:mine/app/mine_num_settings.dart';
import 'package:mine/app/settings_display.dart';
import 'package:mine/display/multi_mine_board_display.dart';
import 'package:mine/game/difficulty.dart';
import 'package:mine/game/multi_mine_game.dart';

class MineApp extends StatelessWidget {
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider<DifficultySettings>(create: (_) => DifficultySettings()),
        ChangeNotifierProvider<MineNumSettings>(create: (_) => MineNumSettings()),
        ChangeNotifierProvider<NeighborSettings>(create: (_) => NeighborSettings()),
        ChangeNotifierProvider<SettingsConfirmation>(create: (_) => SettingsConfirmation()),
      ],
      child: MaterialApp(
        home: MineAppScaffold()
      )
    );
  }
}

class MineAppScaffold extends StatelessWidget {
  final Difficulty config = Difficulty(
    title: "Beginner",
    width: 10,
    height: 10,
    mines: 10
  );

  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: TextButton(
          child: Icon(Icons.settings, color: Colors.white),
          onPressed: () => Navigator.push(
            context,
            MaterialPageRoute(builder: (_) {print("test"); return SettingsDisplay();})
          )
        ),
        title: Text(
          config.toString()
        )
      ),

      body: Center(
        child: MultiMineBoardDisplay(MultiMineGame(config, [1,2]))
      )
    );
  }
}