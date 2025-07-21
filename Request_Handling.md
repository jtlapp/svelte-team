# Sequence Diagram: Request Handling During Play

```mermaid
sequenceDiagram
    actor Browser
    participant Boundary as <<boundary>><br/>SessionController

    # Hydrate the GameSession

    Browser ->> Boundary: (state cookie,<br/>operation, userData)
    create participant GameSession as session<br/>:GameSession
    Boundary ->> GameSession: new (state)
    create participant GameBoard as board<br/>:GameBoard
    GameSession ->> GameBoard: new<br/>(categories,<br/>categoryColors)

    loop for each game square
        create participant Square as :Square
        GameBoard ->> Square: new() or<br/>new (category, color)
    end

    loop for each player
        create participant GamePiece as :GamePiece
        GameSession ->> GamePiece: new (playerName, color, location)
    end

    GameSession --) Boundary: session

    # Perform the Operation

    Boundary ->> GameSession: operation(userData)
    GameSession --) Boundary: result

    # Serialize the game state

    Boundary ->> GameSession: serialize()
    GameSession --) Boundary: state

    Boundary --) Browser: state cookie,<br/>result
