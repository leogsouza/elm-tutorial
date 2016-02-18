# Players Update

We need __src/Players/Update.elm__ to account for the new actions created. Add some imports:

```elm
...
import Task
import Players.Effects exposing (..)
```

Add one branch for `CreatePlayer`:

```elm
    CreatePlayer ->
      ( model.players, create new )
```

This branch returns the `create` effect from `Players.Effects` we added before. `new` creates an empty player, this comes from `Players.Models`. So we return an effect to create a new empty player.

Add another branch for `CreatePlayerDone`:

```elm
...
    CreatePlayerDone result ->
      case result of
        Ok player ->
          let
            updatedCollection =
              player :: model.players

            fx =
              Task.succeed (EditPlayer player.id)
                |> Effects.task
          in
            ( updatedCollection, fx )

        Err error ->
          let
            message =
              toString error

            fx =
              Signal.send model.showErrorAddress message
                |> Effects.task
                |> Effects.map TaskDone
          in
            ( model.players, fx )
```

- `result` will be ` (Result Http.Error Player)` as defined in `Players.Actions`.

- If successful we get `Ok player`, in this case we add the player to the beginning of the collection with ` player :: model.players`.

- Also after we added the player we want to navigate to the edit player view. So we return an effect to do this with `Task.succeed (EditPlayer player.id) |> Effects.task`

- In case of error we get `Err error`. We then follow the pattern we had before to show errors in the main view.
