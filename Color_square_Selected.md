# Sequence Diagram: Color Square Selected

```mermaid
sequenceDiagram
    actor Player
    participant Boundary as <<boundary>><br/>Player
    participant GameSession as session<br/>:GameSession
    participant GamePiece as activePiece<br/>:GamePiece
    participant GameBoard as board<br/>:GameBoard
    participant Square as :ColoredSquare
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
    GameSession ->> GameBoard: getLandingAction(activePiece)
    GameBoard ->> GamePiece: getLocation()
    GamePiece --) GameBoard: location

    %% Player lands on colored square
    GameBoard ->> Square: getLandingAction()
    Square --) GameBoard: DRAW_QUESTION, categoryID
    GameBoard --) GameSession: DRAW_QUESTION, categoryID

    alt action == DRAW_QUESTION
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
            GameBoard ->> Square: markCorrect(activePiece)
            Square --) GameBoard: false
            GameBoard --) GameSession: false
            GameSession --) Boundary: ROLL_AGAIN
            Boundary --) Player: ROLL_AGAIN
        else incorrect answer
            Player ->> Boundary: markIncorrect()
            Boundary ->> GameSession: markIncorrect()
            GameSession --) Boundary: NEXT_PLAYER_ROLLS
            Boundary --) Player: NEXT_PLAYER_ROLLS
        end
    end
