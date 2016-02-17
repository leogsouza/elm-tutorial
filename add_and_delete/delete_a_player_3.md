# Delete a player 3

If the user hits yes we want to send a message back to Elm to delete the player. If they hit cancel we don't send anything and nothing happens.

## Main

Add a new port to __src/Main.elm__ so we can get the inbound message:

```elm
port getDeleteConfirmation : Signal Int
```

And map this port to a signal:

```
getDeleteConfirmationSignal : Signal Actions.Action
getDeleteConfirmationSignal =
  let
    toAction id =
      id
        |> Players.Actions.DeletePlayer
        |> PlayersAction
  in
    Signal.map toAction getDeleteConfirmation
```

Here messages coming from `getDeleteConfirmation` as wrapped as with `Players.Actions.DeletePlayer` first and then `PlayersAction`. 

At the end we finish up with a signal that emits `PlayersAction (Players.Actions.DeletePlayer playerId)`.

This signal needs to be an input to `app`:

```elm
app : StartApp.App AppModel
app =
  StartApp.start
    { init = init
    , inputs = [ routerSignal, actionsMailbox.signal, getDeleteConfirmationSignal ]
    , update = update
    , view = view
    }
 ```
 
This is how __src/Main.elm__ looks at this point <https://github.com/sporto/elm-tutorial-app/blob/0610-delete-player/src/Main.elm>
 
Now we get the message back from JavaScript and we map it to the `DeletePlayer` players action.

## Players Update

Let's add the code in __src/Players/Update.elm__ to respond to this action, add a new branch:

```elm
    DeletePlayer playerId ->
      ( model.players, delete playerId )
```

This returns a `delete` effect we will add next.

## Players Effects

Next, add the effects to delete the player. Add this to __src/Players/Effects.elm__:

```elm
deleteUrl : PlayerId -> String
deleteUrl playerId =
  "http://localhost:4000/players/" ++ (toString playerId)


deleteTask : PlayerId -> Task.Task Http.Error ()
deleteTask playerId =
  let
    config =
      { verb = "DELETE"
      , headers = [ ( "Content-Type", "application/json" ) ]
      , url = deleteUrl playerId
      , body = Http.empty
      }
  in
    Http.send Http.defaultSettings config
      |> Http.fromJson (Decode.succeed ())
```

`deleteTask` takes a player id and returns a task to delete the player.

`Http.send` returns a task of type `Task.Task Http.RawError Http.Response`. But for consistency with other effects we want `Task.Task Http.Error a` where `a` is the parsed JSON.

`Http.fromJson decoder` will parse the returned body into Json and return a type of type `Task.Task Http.Error a` which is what we want. We don't care about the returned body so we use `(Decode.succeed ())` as the decoder. This is a decoder that always succeeds and returns empty. 

So here we are using `Http.fromJson` just for the side effect of converting the task to `Task.Task Http.Error ()`.

In the same file also add:

```elm
delete : PlayerId -> Effects Action
delete playerId =
  deleteTask playerId
    |> Task.toResult
    |> Task.map (DeletePlayerDone playerId)
    |> Effects.task
```

This takes the previous `deleteTask`, converts the result of the task to a `Result` type. Then wraps the result with the `DeletePlayerDone` action and finally converts it to an effect.

## Players Update

