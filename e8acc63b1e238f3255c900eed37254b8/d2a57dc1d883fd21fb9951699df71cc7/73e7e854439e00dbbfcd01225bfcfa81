import 'package:flutter/foundation.dart';
import 'package:mine/game/default_difficulties.dart';
import 'package:mine/game/difficulty.dart';

class DifficultySettings extends ChangeNotifier {
  Difficulty _difficulty = DefaultDifficulties.beginner;

  Difficulty get difficulty => _difficulty;
  set difficulty(Difficulty val) {
    _difficulty = val;
    notifyListeners();
  }
}