# Sequence Diagram: Center square Selected with Less Than 4 Chips

```mermaid
sequenceDiagram
    actor Player
    participant Boundary as <<boundary>><br/>Player
    participant GameSession as session<br/>:GameSession
    participant GamePiece as activePiece<br/>:GamePiece
    participant GameBoard as board<br/>:GameBoard
    participant CenterSquare  as :CenterSquare 
    participant QuestionBank as :QuestionBank

    %% Rolling the die
    Player ->> Boundary: rollDie()
    Boundary ->> GameSession: rollDie()
    GameSession ->> GamePiece: getLocation()
    GamePiece --) GameSession: location
    GameSession ->> GameBoard: getValidMoves(dieRoll, location)
    GameBoard --) GameSession: valid moves
    GameSession --) Boundary: die roll, valid moves
    Boundary --) Player: die roll, valid moves

    %% Selecting a move
    Player ->> Boundary: selectSquare(location)
    Boundary ->> GameSession: selectSquare(location)
    GameSession ->> GamePiece: setLocation(location)
    GamePiece --) GameSession: 

    %% Player lands on center square
    GameSession ->> GameBoard: getLandingAction(location)
    GameBoard ->> CenterSquare : getLandingAction()
    CenterSquare  --) GameBoard: action
    GameBoard --) GameSession: action

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
            alt player has fewer than 4 chips
                GamePiece --) CenterSquare : chips
                CenterSquare  --) GameBoard: false
                GameBoard --) GameSession: false
                GameSession --) Boundary: ROLL_AGAIN
                Boundary --) Player: ROLL_AGAIN
            end
        else incorrect answer
            Player ->> Boundary: markIncorrect()
            Boundary ->> GameSession: markIncorrect()
            GameSession --) Boundary: NEXT_PLAYER_ROLLS
            Boundary --) Player: NEXT_PLAYER_ROLLS
        end
    end
