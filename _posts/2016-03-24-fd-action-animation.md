---
layout: post
title: "Action Animation in Forbidden Desert"
date: 2016-03-24
---

In the last few posts I've discussed how Forbidden Desert uses the Action pattern, with GameAction objects, to organize game changes in the app.

Here I'm going to look at another huge part of the app, and a part that Actions drive: Animation.

The vast majority of the visual changes during a game of FD come from changes in the game itself: A player moves to a new tile, sand piles on a tile, the storm rages across the screen, etc. All those animations are directly tied to changes in the game's state. And all game state changes are directly tied to GameAction objects, so it's natural to use those Actions to organize all the animations (and there are a lot of them).

Here's the basic idea in code:

```swift
class GameViewController: UIViewController {
...
  func doActionAndUpdate(action: GameAction, completion: (() -> ())? = nil) {
    // Change the game state
    action.doAction(self.game) 
    
    // Animate!
    self.animationManager.animateAction(action,
      controller: self,
      game: self.game,
      isUndo: isUndo,
      completion: completion)
  }
}

class AnimationManager {
...
  func animateAction(action: GameAction, controller: GameViewController, game: Game, isUndo: Bool, completion: (() -> ())?) {
    
    let completionOperation = NSBlockOperation(block: { () -> Void in
      if let completionUnwrap = completion {
        dispatch_async(dispatch_get_main_queue()) { completionUnwrap(areThereMoreActions: false)}
      }
    })
    
    if let animateOperation = action.animationOperation(controller, game: game, isUndo: isUndo) {
      completionOperation.addDependency(animateOperation)
      self.animationQueue.addOperation(animateOperation)
    }
    
    self.animationQueue.addOperation(completionOperation)
  }
}
```
