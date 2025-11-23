```markdown
# Raytraced Audio Plugin for Godot

A powerful procedural audio system that adds realistic echo, ambient outdoor sounds, and dynamic muffling to your 3D scenes using raycasting. No need to manually create and maintain audio zones—the plugin automatically adapts to your environment.

**Inspired by:** [Vercidium's amazing video](<https://youtu.be/u6EuAUjq92k?si=6W-sGozYBQITEgQo>) explaining the concept.

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Components](#core-components)
- [Audio Buses](#audio-buses)
- [Configuration Guide](#configuration-guide)
- [Performance Optimization](#performance-optimization)
- [Advanced Usage](#advanced-usage)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Features

### 🎯 Automatic Procedural Effects
- **Dynamic Reverb/Echo**: Automatically calculates room size and applies appropriate reverb based on raycast data
- **Ambient Sound Direction**: Sounds from outside appear to come from openings (doors, windows) and fade based on distance
- **Wall Muffling**: Sounds behind walls are automatically muffled using low-pass filtering
- **No Manual Zones**: Everything is calculated procedurally—just place nodes and it works

### 🎮 Easy Integration
- Drop-in replacement for `AudioListener3D` and `AudioStreamPlayer3D`
- Automatic audio bus setup
- Performance monitoring built-in
- Smart enable/disable system to save resources

---

## Installation

### Manual Installation

1. Download or clone this repository
2. Copy the `addons/raytraced_audio` folder into your project's `addons/` directory
3. Enable the plugin in `Project Settings > Plugins > Raytraced Audio`

### Post-Installation Setup

The plugin will automatically:
- Create two audio buses: `RaytracedReverb` and `RaytracedAmbient`
- Set up project settings for bus names (configurable in `Project Settings > Raytraced Audio`)

**Recommended:** Set `Audio/General/3d_panning_strength` in Project Settings to `1.0` or above for best results.

---

## Quick Start

### Basic Setup (5 minutes)

1. **Add the Listener** (one per scene):

```

In your player/camera scene:

- Add a RaytracedAudioListener node as a child
- Position it at head/ear level (typically ~1.65-1.88 units above ground)
- That's it! Reverb and ambient effects are now active.

```

2. **Use Raytraced Audio Players** (optional, for muffling):

```

Replace AudioStreamPlayer3D with RaytracedAudioPlayer3D

- Sounds will automatically muffle behind walls
- Works with any AudioStreamPlayer3D properties

```

3. **Route Sounds to Audio Buses** (optional):

```

For reverb effects: Set audio bus to "RaytracedReverb"
For ambient effects: Set audio bus to "RaytracedAmbient"

```

**Example Scene Structure:**

```

Player (CharacterBody3D)
├── Head
│   └── Camera3D
└── RaytracedAudioListener  ← Add this

```

---

## Core Components

### RaytracedAudioListener

The main component that performs raycasting and calculates audio effects. **One per scene only** (like `AudioListener3D`).

#### Key Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `is_enabled` | bool | `true` | Enable/disable raycasting |
| `auto_update` | bool | `true` | Update automatically each frame |
| `rays_count` | int | `4` | Number of rays to cast (more = more accurate, slower) |
| `max_raycast_dist` | float | `343.0` | Maximum raycast distance (default: speed of sound in m/s) |
| `max_bounces` | int | `3` | Maximum ray bounces per ray |
| `ray_scatter_model` | enum | `RANDOM` | How rays are distributed (`RANDOM` or `XZ`) |

#### Muffle Settings

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `muffle_enabled` | bool | `true` | Enable muffling for RaytracedAudioPlayer3D nodes |
| `muffle_interpolation` | float | `0.01` | How quickly muffle effect changes (0-1) |

#### Echo/Reverb Settings

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `echo_enabled` | bool | `true` | Enable automatic reverb calculation |
| `echo_room_size_multiplier` | float | `2.0` | Multiplies calculated room size (accounts for sound wave return) |
| `echo_interpolation` | float | `0.01` | How quickly reverb changes (0-1) |

#### Ambient Settings

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ambient_enabled` | bool | `true` | Enable ambient sound direction/panning |
| `ambient_pan_interpolation` | float | `0.02` | How quickly pan direction changes (0-1) |
| `ambient_pan_strength` | float | `1.0` | Panning intensity (0 = no pan, 1 = full pan) |
| `ambient_volume_interpolation` | float | `0.01` | How quickly ambient volume changes |
| `ambient_volume_attenuation` | float | `0.998` | How quickly ambient fades when "inside" (0-1, higher = slower fade) |

#### Read-Only Properties

| Property | Type | Description |
|----------|------|-------------|
| `room_size` | float | Calculated room size in world units |
| `ambience` | float | How "outside" the listener is (0-1, 1 = fully outside) |
| `ambient_dir` | Vector3 | Direction to "outside" based on ray data |
| `ray_casts_this_tick` | int | Performance metric: raycasts this frame |

#### Methods

- `setup()` - Manually initialize rays (called automatically if `is_enabled`)
- `clear()` - Remove all rays
- `update()` - Manually update (only needed if `auto_update = false`)

#### Signals

- `enabled` - Emitted when node is enabled
- `disabled` - Emitted when node is disabled
- `ray_configuration_changed` - Emitted when ray settings change

---

### RaytracedAudioPlayer3D

Drop-in replacement for `AudioStreamPlayer3D` that adds automatic wall muffling.

#### Key Features

- **Automatic Muffling**: Sounds behind walls are automatically low-pass filtered
- **Smart Enable/Disable**: Automatically enables when audible, disables when too far
- **Per-Player Bus**: Creates individual audio bus for each enabled player (for muffling effect)

#### Key Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `audibility_threshold_db` | float | `-30.0` | Volume threshold below which player is disabled |

#### Methods

- `enable()` - Manually enable (usually automatic)
- `disable()` - Manually disable (usually automatic)
- `update(listener: RaytracedAudioListener)` - Update muffling (called automatically)
- `is_audible(from_pos: Vector3) -> bool` - Check if audible from position
- `get_volume_db_from_pos(from_pos: Vector3) -> float` - Get volume at position
- `calculate_audible_distance_threshold() -> float` - Calculate max audible distance

#### Signals

- `enabled` - Emitted when player is enabled
- `disabled` - Emitted when player is disabled
- `audible_distance_updated(distance: float)` - Emitted when max distance changes

#### Important Notes

⚠️ **Bus Assignment**: `RaytracedAudioPlayer3D` defaults to the reverb bus. If you need a different bus, set it **after** the node enters the scene tree (in `_ready()` or later).

---

## Audio Buses

The plugin automatically creates two audio buses:

### RaytracedReverb Bus

**Purpose**: Controls echo/reverb effects

**How it works**:
- The listener calculates room size from raycast data
- Larger enclosed spaces = more reverb
- Open outdoor areas = less reverb
- Reverb parameters (room size, predelay, feedback) are automatically adjusted

**Usage**:
- Route sounds that should have reverb to this bus
- Example: Footsteps, voice, weapons firing

### RaytracedAmbient Bus

**Purpose**: Controls ambient outdoor sounds with directional panning

**How it works**:
- Detects "escape routes" (openings to outside) via raycasting
- Sounds in this bus appear to come from the direction of openings
- Volume fades based on distance to openings
- Panning automatically adjusts based on opening direction

**Usage**:
- Route outdoor ambient sounds to this bus
- Example: Wind, distant traffic, nature sounds, outdoor music

### Customizing Bus Names

Change bus names in `Project Settings > Raytraced Audio`:
- `raytraced_audio/reverb_bus` (default: `"RaytracedReverb"`)
- `raytraced_audio/ambient_bus` (default: `"RaytracedAmbient"`)

---

## Configuration Guide

### Performance vs Quality Trade-offs

#### Ray Count (`rays_count`)

- **Low (2-4)**: Fast, less accurate, good for simple scenes
- **Medium (4-8)**: Balanced, recommended for most games
- **High (8-16)**: Slow, very accurate, use for cinematic experiences

**Formula**: Actual raycasts per frame = `rays_count * (2 + n)` where `n` = enabled `RaytracedAudioPlayer3D` count

#### Max Bounces (`max_bounces`)

- **1-2**: Fast, basic reverb detection
- **3-4**: Balanced, recommended
- **5+**: Slow, very detailed reverb, rarely needed

#### Max Raycast Distance (`max_raycast_dist`)

- **Default (343.0)**: Speed of sound in m/s, realistic
- **Lower**: Faster, but may miss distant walls
- **Higher**: Slower, but detects larger spaces

### Ray Scatter Models

#### RANDOM (Default)
- Rays cast in random 3D directions
- Best for general use, detects all directions equally
- Use for: Most scenarios

#### XZ
- Rays cast only on horizontal plane (XZ plane)
- Faster, but ignores vertical geometry
- Use for: Top-down games, 2.5D games, or when vertical audio isn't important

### Interpolation Settings

All interpolation values control how quickly effects change:
- **Lower (0.001-0.01)**: Smooth, gradual changes (recommended)
- **Higher (0.1-1.0)**: Snappy, immediate changes (may feel jarring)

**Recommended values**:
- `muffle_interpolation`: `0.01`
- `echo_interpolation`: `0.01`
- `ambient_pan_interpolation`: `0.02`
- `ambient_volume_interpolation`: `0.01`

---

## Performance Optimization

### Understanding Performance Cost

The plugin's performance depends on:
1. **Ray count** (`rays_count`)
2. **Max bounces** (`max_bounces`)
3. **Number of enabled `RaytracedAudioPlayer3D` nodes**
4. **Scene complexity** (collision geometry)

**Actual raycast count per frame**:

```

rays_count * (2 + enabled_players_count)

```

Where:
- `1` = Main ray that bounces around
- `1` = Echo detection ray
- `enabled_players_count` = Muffle rays (one per enabled player)

### Performance Monitors

The plugin adds two performance monitors (view in `Debugger > Monitors`):

1. **`raytraced_audio/raycast_updates`**: Total raycasts this frame
2. **`raytraced_audio/enabled_players_count`**: Number of enabled audio players

### Optimization Tips

1. **Reduce Ray Count**: Start with 4, increase only if needed
2. **Limit Bounces**: 3 bounces is usually sufficient
3. **Use XZ Scatter**: If vertical audio isn't important, use `XZ` scatter model
4. **Reduce Max Distance**: If your rooms are small, reduce `max_raycast_dist`
5. **Limit Audio Players**: Each enabled player adds raycasts
6. **Manual Updates**: Set `auto_update = false` and call `update()` manually (e.g., every 2-3 frames)
7. **Disable When Not Needed**: Set `is_enabled = false` when audio effects aren't needed

### Example: Manual Update (Every 2 Frames)

```gdscript
# In your RaytracedAudioListener script
var frame_count: int = 0

func _process(delta: float) -> void:
    frame_count += 1
    if frame_count % 2 == 0:  # Update every 2 frames
        update()

```

---

## Advanced Usage

### Manual Update Control

For fine-grained control, disable auto-update and call manually:

```
# In your scene
@onready var audio_listener: RaytracedAudioListener = $RaytracedAudioListener

func _ready():
    audio_listener.auto_update = false

func _process(delta):
    # Update only when player moves
    if player_has_moved:
        audio_listener.update()

```

### Custom Bus Assignment

```
# For RaytracedAudioPlayer3D
func _ready():
    # Must set bus after _enter_tree()
    bus = "MyCustomBus"

```

### Listening to Signals

```
func _ready():
    var listener = $RaytracedAudioListener
    listener.ray_configuration_changed.connect(_on_config_changed)
    listener.enabled.connect(_on_listener_enabled)

    var player = $RaytracedAudioPlayer3D
    player.enabled.connect(_on_player_enabled)
    player.audible_distance_updated.connect(_on_distance_updated)

func _on_config_changed():
    print("Ray configuration changed!")

func _on_listener_enabled():
    print("Listener enabled")

func _on_player_enabled():
    print("Audio player enabled")

func _on_distance_updated(distance: float):
    print("Max audible distance: ", distance)

```

### Accessing Calculated Data

```
func _process(delta):
    var listener = $RaytracedAudioListener
    var room_size = listener.room_size
    var ambience = listener.ambience  # 0 = inside, 1 = outside
    var outside_dir = listener.ambient_dir

    # Use data for gameplay logic
    if ambience > 0.5:
        print("Player is mostly outside")

```

### Dynamic Configuration

```
# Adjust settings based on performance
func _process(delta):
    var raycast_count = Performance.get_monitor(Performance.CUSTOM + 0)  # Adjust index
    if raycast_count > 100:  # Too many raycasts
        $RaytracedAudioListener.rays_count = max(2, $RaytracedAudioListener.rays_count - 1)
    elif raycast_count < 20:  # Can afford more
        $RaytracedAudioListener.rays_count = min(8, $RaytracedAudioListener.rays_count + 1)

```

---

## Best Practices

### Scene Setup

1. **One Listener Per Scene**: Never use multiple `RaytracedAudioListener` nodes
2. **Position at Head Level**: Place listener at ear/head height (~1.65-1.88 units)
3. **Parent to Player/Camera**: Attach to the same node as your camera

### Audio Player Usage

1. **Use for Important Sounds**: Use `RaytracedAudioPlayer3D` for sounds that benefit from muffling (footsteps, voice, weapons)
2. **Regular Players for Background**: Use regular `AudioStreamPlayer3D` for background music/ambient (route to ambient bus if needed)
3. **Set Max Distance**: Configure `max_distance` on players to limit processing

### Bus Routing Strategy

- **Reverb Bus**: Sounds that should echo (footsteps, voice, weapons, impacts)
- **Ambient Bus**: Outdoor ambient sounds (wind, traffic, nature)
- **Master Bus**: Music, UI sounds, non-spatial audio

### Collision Setup

- **Ensure Collision Layers**: Make sure walls/floors have collision shapes
- **Layer 1 Recommended**: Use collision layer 1 for geometry (default)
- **Avoid Overlapping Geometry**: Can cause raycast issues

### Performance Management

1. **Start Conservative**: Begin with `rays_count = 4`, `max_bounces = 3`
2. **Profile Regularly**: Check performance monitors
3. **Adjust Per Scene**: Different scenes may need different settings
4. **Disable When Paused**: Set `is_enabled = false` in pause menus

---

## Troubleshooting

### No Reverb/Echo Effects

**Check:**

- Is `echo_enabled = true`?
- Is the listener `is_enabled = true`?
- Are sounds routed to `RaytracedReverb` bus?
- Does the scene have collision geometry?

**Solution:**

- Verify collision shapes exist on walls/floors
- Check that reverb bus exists in Audio tab
- Ensure listener is updating (check `ray_casts_this_tick > 0`)

### No Muffling

**Check:**

- Are you using `RaytracedAudioPlayer3D` (not regular `AudioStreamPlayer3D`)?
- Is `muffle_enabled = true` on the listener?
- Is the player enabled? (check `is_enabled()`)
- Are there walls with collision between listener and player?

**Solution:**

- Replace `AudioStreamPlayer3D` with `RaytracedAudioPlayer3D`
- Verify collision layers are set correctly
- Check that player is within `max_distance`

### Ambient Sounds Not Working

**Check:**

- Is `ambient_enabled = true`?
- Are sounds routed to `RaytracedAmbient` bus?
- Is the listener detecting "outside" (`ambience > 0`)?

**Solution:**

- Ensure there are openings (doors, windows) in your geometry
- Check `ambient_dir` to see if direction is being calculated
- Verify ambient bus exists and has volume > 0

### Performance Issues

**Symptoms:**

- Low FPS when plugin is active
- High `raycast_updates` count

**Solutions:**

1. Reduce `rays_count` (try 2-4)
2. Reduce `max_bounces` (try 2-3)
3. Use `XZ` scatter model if vertical audio isn't needed
4. Reduce `max_raycast_dist` if rooms are small
5. Limit number of `RaytracedAudioPlayer3D` nodes
6. Enable manual updates with lower frequency
7. Disable when not needed (`is_enabled = false`)

### Bus Not Found Errors

**Check:**

- Is plugin enabled in Project Settings?
- Did you change bus names? Verify in `Project Settings > Raytraced Audio`

**Solution:**

- Re-enable plugin to recreate buses
- Or manually create buses with correct names

### Audio Players Not Enabling

**Check:**

- Is player within `max_distance`?
- Is `audibility_threshold_db` too high?
- Is player actually playing (`playing = true`)?

**Solution:**

- Increase `max_distance` or decrease `audibility_threshold_db`
- Ensure audio stream is set and playing

---

## Technical Details

### How It Works

1. **Raycasting**: The listener casts rays in random directions
2. **Bouncing**: Rays bounce off walls, gathering information about the environment
3. **Echo Detection**: Rays that return to the listener calculate room size for reverb
4. **Escape Detection**: Rays that escape detect "outside" for ambient effects
5. **Muffling**: Additional rays check line-of-sight to audio players for muffling

### Ray Types

Each ray performs multiple functions:

- **Main Ray**: Bounces around, detects room geometry
- **Echo Ray**: Checks if path back to listener is clear (for reverb calculation)
- **Muffle Ray**: Checks line-of-sight to each enabled audio player (for muffling)

### Audio Bus Architecture

```
Master Bus
├── RaytracedReverb (with Reverb effect)
│   └── Individual Player Buses (with LowPass for muffling)
│       └── RaytracedAudioPlayer3D nodes
└── RaytracedAmbient (with Panner effect)
    └── Ambient sound sources

```

---

## License

See `LICENSE` file in the plugin directory.

---

## Credits

- **Author**: Tienne_k
- **Concept Inspiration**: [Vercidium's Video](https://youtu.be/u6EuAUjq92k?si=6W-sGozYBQITEgQo)

---

## Support

For issues, questions, or contributions, please refer to the main project repository.

```

This documentation includes:
- Overview and features
- Installation and setup
- Component reference
- Configuration options
- Performance guidance
- Advanced usage examples
- Best practices
- Troubleshooting
- Technical details

It should help developers use the plugin effectively.
```