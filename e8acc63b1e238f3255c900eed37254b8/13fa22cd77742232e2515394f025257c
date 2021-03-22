import 'dart:math';

import 'package:flutter/material.dart';

class SquaresGrid extends StatelessWidget {
  final int width;
  final int height;

  // takes a point
  final Function generateWidget;

  SquaresGrid({required this.width, required this.height, required this.generateWidget});

  Widget build(BuildContext context) {
    Widget _generateRow(int y) {
      List<Widget> children = List.generate(
        width,
        (int x) => Expanded(child: generateWidget(Point(x, y)))
      );
      return Row(children: children, crossAxisAlignment: CrossAxisAlignment.stretch);
    }

    var colChildren = List.generate(height, (int y) => Expanded(child: _generateRow(y)));

    return AspectRatio(
      child: Column(
        children: colChildren
      ),
      aspectRatio: width / height,
    );
  }
}