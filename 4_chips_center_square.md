# Sequence Diagram: Center Square Selected with 4 Chips

```mermaid
sequenceDiagram
    actor Player
    participant Boundary as <<boundary>><br/>Player
    participant GameSession as session<br/>:GameSession
    participant GamePiece as activePiece<br/>:GamePiece
    participant GameBoard as board<br/>:GameBoard
    participant CenterSquare  as :CenterSquare 
    participant QuestionBank as :QuestionBank

    %% Selecting a move
    Player ->> Boundary: rollDie()
    Boundary ->> GameSession: rollDie()
    GameSession ->> GamePiece: getLocation()
    GamePiece --) GameSession: location

    %% Player lands on center CenterSquare 
    GameSession ->> GameBoard: getValidMoves(dieRoll, location)
    GameBoard --) GameSession: valid moves
    GameSession --) Boundary: die roll, valid moves
    Boundary --) Player: die roll, valid moves
    Player ->> Boundary: selectSquare(location)
    Boundary ->> GameSession: selectSquare(location)


    %% GameSession delegates location update
    GameSession->>GamePiece: setLocation(location)

    %% Get the selected square
    GameSession->>GameBoard: getLandingAction(activePiece)
    GameBoard ->> GamePiece: getLocation()
    GamePiece --) GameBoard: location
    GameBoard ->> CenterSquare : getLandingAction()
    CenterSquare  --) GameBoard: PROVIDE_CATEGORY, categoryID
    GameBoard --) GameSession: PROVIDE_CATEGORY, categoryID
    
    %% Answer Question
    alt action == PROVIDE_CATEGORY
        GameSession --) Boundary: PROVIDE_CATEGORY
        Boundary --) Player: PROVIDE_CATEGORY
        Player ->> Boundary: getQuestion(category)
        Boundary ->> GameSession: getQuestion(category)
        GameSession ->> QuestionBank: getQuestionIDs(category)
        QuestionBank --) GameSession: question IDs
        GameSession ->> QuestionBank: getQuestion(questionID)
        QuestionBank --) GameSession: question/answer
        GameSession --) Boundary: question/answer
        Boundary --) Player: question/answer
    
    alt correct answer
            Player ->> Boundary: markCorrect()
            Boundary ->> GameSession: markCorrect()
            GameSession ->> GameBoard: markCorrect(activePiece)
            GameBoard ->> GamePiece: getLocation()
            GamePiece --) GameBoard: location
            GameBoard ->> CenterSquare : markCorrect(activePiece)
            CenterSquare  ->> GamePiece: getChips()
            alt player has 4 chips
                GamePiece --) CenterSquare : chips
                CenterSquare  --) GameBoard: true
                GameBoard --) GameSession: true
                GameSession --) Boundary: PLAYER_WINS
                Boundary --) Player: PLAYER_WINS
            end
        else incorrect answer
            Player ->> Boundary: markIncorrect()
            Boundary ->> GameSession: markIncorrect()
            GameSession --) Boundary: NEXT_PLAYER_ROLLS
            Boundary --) Player: NEXT_PLAYER_ROLLS
        end
    end
