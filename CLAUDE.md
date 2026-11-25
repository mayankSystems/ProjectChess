# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Project Chess is a Java-based chess game with a Swing/AWT GUI. It implements standard chess rules including:
- All 6 piece types with correct movement rules
- Check/checkmate detection
- Stalemate detection
- Castling (kingside and queenside)
- En passant capture
- Pawn promotion

## Build System

**Maven** is used for building and dependency management.

### Common Commands

```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Package as JAR (with dependencies)
mvn clean package

# Run the application
java -cp target/ProjectChess-v1.0.0-jar-with-dependencies.jar dev.mayank.app.ChessApp

# Or run the pre-built executable
./ProjectChess-1.0.exe
```

### Project Structure

```
src/main/java/dev/mayank/
├── app/
│   ├── ChessApp.java       # Entry point - creates JFrame and GamePanel
│   ├── ChessBoard.java     # Board rendering (8x8 grid, square colors)
│   ├── GamePanel.java      # Main game loop, state management, input handling
│   └── Mouse.java          # Mouse event handling (drag, click, release)
└── piece/
    ├── ChessPiece.java     # Abstract base class - position, movement validation helpers
    ├── Pawn.java           # Pawn movement (1/2 step, capture, en passant, promotion)
    ├── Rook.java           # Rook movement (horizontal/vertical, no jumping)
    ├── Knight.java         # Knight movement (L-shape)
    ├── Bishop.java         # Bishop movement (diagonal)
    ├── Queen.java          # Queen movement (rook + bishop)
    └── King.java           # King movement (1 square + castling logic)
```

## Architecture Notes

### Game Loop
- `GamePanel` implements `Runnable` and runs at 60 FPS
- `update()` handles game state changes (piece selection, movement simulation, validation)
- `paintComponent()` handles rendering (board, pieces, highlights, status messages)

### Piece Movement Logic
Each piece extends `ChessPiece` and implements `canMove(targetRow, targetCol)`:
- Uses `prevRow`/`prevCol` for validation (position before current drag)
- `simPieces` list used for move simulation without mutating actual board state
- `isValidNextMove()` checks destination occupancy (empty or opponent piece)
- Path checking: `isPieceOnStraightPath()` and `isPieceOnDiagonalPath()`

### Special Rules
- **Castling**: Handled in `King.canMove()` via `canKingCastle()` - checks rook position, path clearance, and move legality
- **En passant**: Handled in `Pawn.canMove()` - detects opponent pawn that just moved 2 squares
- **Promotion**: Handled in `GamePanel` - when pawn reaches last rank, shows promotion options
- **Check/Checkmate**: `isKingInCheck()`, `isCheckMate()`, `simulateKingMoves()` in `GamePanel`
- **Stalemate**: `isStalemateCondition()` in `GamePanel`

### State Management
- `pieces` - authoritative list of pieces (committed state)
- `simPieces` - simulation list (modified during drag, reverted on invalid move)
- `activePiece` - currently selected/dragged piece
- `currentColor` - whose turn (WHITE=0, BLACK=1)

## Testing

Tests use **JUnit 3.8.1** (legacy). Test file: `src/test/java/dev/mayank/ChessAppTest.java`

Run single test:
```bash
mvn test -Dtest=ChessAppTest
```

Note: The test file currently contains only a placeholder test and a commented-out test for illegal king moves.

## Key Files to Understand

1. **`GamePanel.java`** - Core game logic (600+ lines): game loop, move validation, checkmate/stalemate, castling, promotion
2. **`ChessPiece.java`** - Base class with shared movement utilities (path checking, coordinate conversion, hit detection)
3. **Piece subclasses** - Each implements `canMove()` with piece-specific rules
4. **`ChessBoard.java`** - Simple board rendering

## Development Notes

- Square size: 100x100 pixels (`ChessBoard.SQUARE_SIZE`)
- Board: 8x8 squares (`ChessBoard.MAX_ROWS`, `MAX_COLS`)
- Panel size: 1100x800 (board + side panel for status)
- Images loaded from `src/main/resources/images/piece/` (e.g., `w-pawn.png`, `b-king.png`)
- No external dependencies beyond JDK (Swing/AWT) and JUnit

## Adding Features

To add a new piece type:
1. Create new class in `dev.mayank.piece` extending `ChessPiece`
2. Implement `canMove()` with movement logic
3. Add piece images to resources
4. Register in `GamePanel.setPieces()` for initial board setup