# Bubble Twist - Technical Implementation Details

This document provides a comprehensive technical overview of the Bubble Twist game implementation, covering architecture, frameworks, and key systems for developers working on or contributing to the project.

## Technology Stack

### Core Frameworks
- **Flutter**: Cross-platform app development framework
- **Flame**: Game engine built on top of Flutter (version 0.12.0)
- **Dart**: Programming language for all game logic and UI

### Key Dependencies
- `flame: ^0.12.0` - Game engine for rendering and game loop
- `shared_preferences: ^0.4.2` - Local data persistence
- `sqflite: ^1.1.0` - Local database for high scores
- `firebase_admob: ^0.8.0+4` - Advertisement integration
- `http: ^0.12.0+1` - Network requests for leaderboards
- `crypto: ^2.0.6` - Cryptographic functions
- `vector_math: any` - Mathematical calculations for physics

## Architecture Overview

### Game Structure
```
Game (BaseGame)
├── Bubbles - Manages bubble entities and game field
├── Player - Handles input, shooting, and player state
├── FloatingNumbers - Visual score feedback system
└── GameWrapper - Flutter widget integration
```

### Entity Management

#### Bubble System
- **Bubble Class**: Core entity representing individual bubbles
- **Anchor System**: Central anchor point around which bubbles rotate
- **Color System**: 6 predefined colors (0-5 integer mapping)
- **Grid Layout**: Hexagonal positioning for optimal bubble arrangement

#### Key Bubble Properties
```dart
class Bubble {
  double x, y;        // Position coordinates
  int color;          // Color identifier (0-5, -1 for anchor)
  double br;          // Bubble radius (static)
  Sprite sprite;      // Visual representation
}
```

### Physics and Collision System

#### Rotation Mechanics
- **Rotation Anchor**: All bubbles rotate around a central `Ancor` object
- **Rotation Calculation**: Uses `DamienMath.rotatePoint()` for mathematical rotation
- **Rotational Velocity**: Dynamic rotation speed based on game events

#### Collision Detection
- **Distance-Based**: Uses `DamienMath.distanceBetween()` for bubble proximity
- **Hit Detection**: Identifies bubble collisions during shooting trajectory
- **Attachment Logic**: Determines valid bubble placement positions

#### Trajectory System
- **Shooting Path**: Calculates bubble flight path from spawn point
- **Wall Collision**: Handles bouncing off game boundaries
- **Speed Control**: Constant bubble speed defined by `bSpeed`

### Level Data and Generation

#### Procedural Generation
```dart
void generateGame() {
  final int layers = 6;
  // Creates hexagonal pattern of bubbles
  // Each layer contains i * 6 bubbles arranged in circle
  // Additional bubbles fill gaps between layers
}
```

#### Level Structure
- **Layers**: 6 concentric layers of bubbles around anchor
- **Hexagonal Pattern**: Optimal packing for bubble arrangement
- **Random Colors**: Bubbles assigned random colors from available set
- **Color Availability**: Only colors present in current formation are used for new bubbles

### Game State Management

#### Core Game States
- **Playing**: Normal gameplay state
- **Game Over**: Triggered when strikes depleted
- **Menu**: Paused state with menu overlay
- **Reset**: Clean slate for new game

#### State Persistence
```dart
// Score tracking
int score = 0;
int multiplier = 1;
int strikes = 6;
int nextStrikes = 5;

// Game progression
bool gameoverCalled = false;
bool fired = false;
```

### Bubble Logic and Matching

#### Pop Detection Algorithm
```dart
bool doPops(Bubble b) {
  // Breadth-first search for connected bubbles
  // Checks all 6 hexagonal directions
  // Pops groups of 3+ matching colors
}
```

#### Floating Bubble Removal
```dart
void removeFloating() {
  // Identifies bubbles not connected to anchor
  // Uses graph traversal from anchor point
  // Removes disconnected bubble clusters
}
```

### User Interface Layer

#### Flutter Integration
- **GameWrapper**: StatefulWidget that hosts the Flame game
- **Overlay System**: Flutter widgets rendered over game canvas
- **Dialog System**: Modal dialogs for menus and settings
- **Gesture Detection**: Touch input handling for shooting and aiming

#### UI Components
- **CircleButton**: Reusable circular button widget
- **Score Display**: Real-time score updates via setState()
- **Menu System**: Tabbed interface for different game sections

### Data Persistence

#### SharedPreferences Storage
```dart
// User account data
static final String _username = "USERNAME";
static final String _id = "ID";
static final String _pass = "PASS";

// Activity tracking
static final String _lastClearedDaily = "lastClearedDaily";
static final String _lastClearedWeekly = "lastClearedWeekly";
static final String _lastClearedMonthly = "lastClearedMonthly";

// Privacy settings
static final String leaderboardConcent = "leaderboardConcent";
```

#### SQLite Database Schema
```sql
-- High score tables
CREATE TABLE allTime (score INTEGER);
CREATE TABLE daily (score INTEGER);
CREATE TABLE weekly (score INTEGER);
CREATE TABLE monthly (score INTEGER);
```

#### Data Management
- **Automatic Cleanup**: Periodic cleaning of time-based leaderboards
- **Duplicate Prevention**: Prevents duplicate score entries
- **Privacy Compliance**: Consent-based data collection

### Network and Leaderboards

#### Global Leaderboard System
- **HTTP Requests**: Communication with remote leaderboard server
- **WebHelper Class**: Abstraction for web service communication
- **Privacy Controls**: User consent required for score submission

#### Friend Code System
```dart
static Future<String> getFriendCode() async {
  final id = await getID();
  final pass = await getPass();
  int key = int.parse(pass.substring(pass.length - 3));
  int friendCode = (id * 737 - key) * 1000 + key;
  return friendCode.toString();
}
```

### Rendering System

#### Flame Rendering Pipeline
- **Canvas API**: Direct Canvas drawing for game elements
- **Sprite Rendering**: Image-based bubble representations
- **Layer Management**: Proper z-ordering of game elements

#### Visual Elements
```dart
@override
void render(Canvas c) {
  // Background
  Paint p = new Paint();
  p.color = backgroundColor;
  c.drawRect(Rect.fromLTWH(0, 0, size.width, size.height), p);
  
  // Walls
  walls.forEach((w) => c.drawRect(w, p));
  
  // Game entities
  bubbles.render(c);
  player.render(c);
  floatingNumbers.render(c);
}
```

### Input Handling

#### Touch Input Processing
- **Gesture Detection**: Uses Flutter's GestureDetector
- **Drag Updates**: Continuous tracking for aiming
- **Tap Events**: Shooting action trigger
- **Coordinate Conversion**: Screen to game coordinate mapping

#### Player Interaction Flow
1. **Drag**: Update aiming angle based on touch position
2. **Tap**: Fire bubble in aimed direction
3. **Collision**: Detect bubble placement and trigger effects
4. **State Update**: Update game state based on results

### Animation and Visual Effects

#### Floating Numbers System
```dart
class FloatingNumber {
  num x, y, value, alpha;
  
  tick() {
    alpha -= 2;           // Fade out
    if (alpha < 150) alpha -= 5;  // Accelerated fade
    y -= 1.5;             // Float upward
  }
}
```

#### Sprite Management
- **Preloaded Sprites**: All bubble sprites loaded at game start
- **Color Mapping**: Integer to sprite mapping for bubble types
- **Asset Organization**: Structured asset loading from `/assets/images/`

### Platform Integration

#### Multi-Platform Support
- **Android**: Native Android app via Flutter
- **iOS**: Native iOS app via Flutter
- **Web**: Web app deployment via Flutter Web

#### Platform-Specific Features
- **Orientation Lock**: Portrait mode enforcement on mobile
- **Ad Integration**: Firebase AdMob for revenue
- **URL Launching**: External link handling for privacy policy

### Performance Considerations

#### Game Loop Optimization
- **Fixed Timestep**: 15ms update intervals (66.7 FPS target)
- **Efficient Rendering**: Minimal overdraw and state changes
- **Memory Management**: Proper cleanup of game entities

#### Resource Management
- **Sprite Caching**: Reuse of sprite instances
- **List Management**: Efficient add/remove operations for bubble collections
- **Database Optimization**: Indexed queries and connection management

### Development Tools and Testing

#### Debug Features
- **Console Logging**: Debugging output for development
- **State Inspection**: Game state visibility for testing
- **Performance Monitoring**: Frame rate and update timing

#### Code Organization
- **Modular Design**: Separate files for different game systems
- **Helper Classes**: Utility classes for math, preferences, and web operations
- **Clean Architecture**: Separation of game logic from UI code

### Extension Points

#### Adding New Features
- **Power-ups**: Extend Bubble class with special behavior types
- **Sound System**: Integrate audio using Flame's audio capabilities
- **Particle Effects**: Add visual effects using Flame's particle system
- **New Bubble Types**: Extend color system and add special bubble behaviors

#### Customization Options
- **Difficulty Scaling**: Modify generation parameters for harder levels
- **Color Schemes**: Alternative bubble color sets for accessibility
- **Game Modes**: Additional play modes beyond endless progression
- **Achievement System**: Track and reward specific player accomplishments