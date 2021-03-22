import 'dart:math';

import 'package:flutter/material.dart';
import 'package:mine/app/neighbor_settings.dart';
import 'package:provider/provider.dart';

class NeighborSelection extends StatelessWidget {
  static final List<int> coords = [-2, -1, 0, 1, 2];

  Widget build(BuildContext context) {
    return Consumer<NeighborSettings>(
      builder: (_, settings, __) => AspectRatio(
        child: Column(
          children: coords.map((y) => Expanded(
            child: FittedBox(
              child: ToggleButtons(
                children: coords.map((x) => x != 0 || y != 0 ? Text("($x, $y)") : Text("ORIGIN")).toList(),
                isSelected: coords.map((x) => settings.neighbors.contains(Point(x, y))).toList(),
                onPressed: (i) => i - 2 != 0 || y != 0 ? settings.toggle(Point(i - 2, y)) : null
              ),
              fit: BoxFit.contain
            )
          )).toList(),

          //crossAxisAlignment: CrossAxisAlignment.stretch
        ),
        aspectRatio: 1
      )
    );
  }
}