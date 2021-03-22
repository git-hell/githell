import 'package:flutter/material.dart';

class SelectorSquare extends StatelessWidget {
  final Widget child;
  final bool isSelected;
  final Function onTap;

  final Color unselectedBgColor = Colors.grey;
  final Color selectedBgColor = Colors.green;

  SelectorSquare({required this.child, required this.isSelected, required this.onTap});

  Widget build(BuildContext context) {
    return GestureDetector(
      child: Container(
        child: child,
        color: isSelected ? selectedBgColor : unselectedBgColor
      ),
      onTap: onTap(),
      behavior: HitTestBehavior.translucent
    );
  }
}