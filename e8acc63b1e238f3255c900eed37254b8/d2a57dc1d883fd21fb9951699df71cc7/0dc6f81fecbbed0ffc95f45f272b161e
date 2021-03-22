import 'package:flutter/material.dart';
import 'package:mine/app/difficulty_selection.dart';
import 'package:mine/app/mine_num_selection.dart';
import 'package:mine/app/neighbor_selection.dart';

class SettingsDisplay extends StatelessWidget {
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: 3,
      child: Scaffold(
        appBar: AppBar(
          title: Text("Settings"),
          bottom: TabBar(tabs: [
            Tab(child: Text("Difficulty")),
            Tab(child: Text("Mines")),
            Tab(child: Text("Neighbors"))
          ])
        ),
        body: TabBarView(
          children: [
            DifficultySelection(),
            Center(child: MineNumSelection()),
            Center(child: NeighborSelection())
          ]
        )
      )
    );
  }
}