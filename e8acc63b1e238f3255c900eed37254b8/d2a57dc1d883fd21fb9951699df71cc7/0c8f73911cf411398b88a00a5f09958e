import 'package:flutter/material.dart';
import 'package:mine/app/mine_num_settings.dart';
import 'package:provider/provider.dart';

class MineNumSelection extends StatelessWidget {
  int _coordToNum(int x, int y) {
    return y * 3 + x + 1;
  }

  Widget build(BuildContext context) {
    return Consumer<MineNumSettings>(
      builder: (context, settings, _) => AspectRatio(
        child: Column(
          children: List.generate(3, (y) => Expanded(
            child: FittedBox(
              child: ToggleButtons(
                children: List.generate(3, (x) => Text((_coordToNum(x, y)).toString())),
                onPressed: (x) => settings.toggle(_coordToNum(x, y)),
                isSelected: List.generate(3, (x) => settings.mineNums.contains(_coordToNum(x, y)))
              ),
              fit: BoxFit.contain
            )
          )),
          //crossAxisAlignment: CrossAxisAlignment.stretch
        ),
        aspectRatio: 1
      )
    );
  }
}