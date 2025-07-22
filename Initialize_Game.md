# Sequence Diagram: Players Initialize Trivial Compute

```mermaid
sequenceDiagram
    actor Browser
    participant Boundary as <<boundary>><br/>SessionController

    %% Create a GameSession
    Browser ->> Boundary: startGame()
    create participant GameSession as session<br/>:GameSession
    Boundary ->> GameSession: new ()
    GameSession --) Boundary: session
    Boundary ->> QuestionBank: getCategories()
    QuestionBank --) Boundary: categories
    participant QuestionBank as :QuestionBank

    %% Serialize Initial GameSession
    Boundary ->> GameSession: serialize()
    GameSession --) Boundary: state
    Boundary --) Browser: state cookie, categories

    %% Users Select Categories
    loop 4 times
        Browser ->> Boundary: selectCategory(categoryID, color)
        Boundary ->> GameSession: selectCategory(categoryID, color)
        GameSession --) Boundary: 
        Boundary --) Browser: 
    end

    %% Add Players
    loop 1 to 4 times
        Browser ->> Boundary: addPlayer(name, color)
        Boundary ->> GameSession: addPlayer(name, color)

        create participant GamePiece as piece<br/>:GamePiece
        GameSession ->> GamePiece: new (playerName, color)
        GamePiece --) GameSession: piece
        GameSession --) Boundary: 
        Boundary --) Browser: 
    end
