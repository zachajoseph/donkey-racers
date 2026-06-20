# INTERFACES.md

This document defines every topic, message type, and publish rate used across the parking capstone. It is the contract between all four tracks.

**This file is locked.** If your track needs a change to an interface, open a PR against this file and tag the other tracks affected. Do not change a message type or topic name inside your own node without updating this doc first — the whole point of this file is that no one has to read another track's code to know what to expect from it.

---

## How to read this

Each topic entry includes:
- **Topic name** — the exact string used in `create_publisher` / `create_subscription`
- **Message type** — the ROS2 message type, all standard types from `std_msgs`, `sensor_msgs`, and `geometry_msgs`
- **Publisher** — which track's node publishes this
- **Subscriber(s)** — which track's node(s) read this
- **Rate** — how often it's expected to publish
- **Fields that matter** — the specific fields downstream nodes actually use

---

## Track 1 → Track 2: Perception → RL Agent

### `/camera/image_raw`
- **Type:** `sensor_msgs/Image`
- **Publisher:** Perception (`camera_node`)
- **Subscriber:** Perception (`cv_state_node`, internal to Track 1)
- **Rate:** 20 Hz
- **Notes:** Raw RPi camera feed. Not consumed outside Track 1 — `cv_state_node` is the only subscriber. Included here for completeness since it's the root of the whole perception pipeline.

### `/parking/state`
- **Type:** `std_msgs/Float32MultiArray`
- **Publisher:** Perception (`cv_state_node`)
- **Subscriber:** RL Agent (`rl_agent_node`)
- **Rate:** 10 Hz
- **Fields that matter:** A 5-element array, in this fixed order:
  1. `slot_visible` — 0, 1, or 2 (which slot is in view, -1 if none)
  2. `alignment` — -1 (left), 0 (center), 1 (right)
  3. `distance_coarse` — 0 (far), 1 (mid), 2 (close)
  4. `in_slot` — 0 or 1
  5. `angle_error_deg` — float, -90 to 90

---

## Track 4 → Track 2: Fleet Networking → RL Agent

### `/fleet/state_in`
- **Type:** `std_msgs/String`
- **Publisher:** Fleet Networking (`net_node`)
- **Subscriber:** RL Agent (`rl_agent_node`)
- **Rate:** 5 Hz (or on-change, whichever is more frequent)
- **Fields that matter:** JSON-encoded string with keys:
  - `assigned_slot` — int, the slot this car has been assigned by the coordinator
  - `slot_occupied_by_other` — bool, true if another car currently holds the target slot
  - `car_id` — string, this car's own ID (for logging/debug)

### `/fleet/state_out`
- **Type:** `std_msgs/String`
- **Publisher:** RL Agent (`rl_agent_node`)
- **Subscriber:** Fleet Networking (`net_node`)
- **Rate:** 5 Hz
- **Fields that matter:** JSON-encoded string with keys:
  - `car_id` — string
  - `current_state` — same 5-field array as `/parking/state`, passed through
  - `requesting_slot` — int or null, set when the agent wants to claim a new slot

---

## Track 4 ↔ Coordinator Server: Fleet Networking → HTTP

*(Not a ROS topic — this is the REST interface between `net_node` and the Flask coordinator server running on a laptop.)*

### `POST /claim`
- **Caller:** Fleet Networking (`net_node`)
- **Body:** `{ "car_id": str, "requested_slot": int }`
- **Response:** `{ "assigned_slot": int, "status": "confirmed" | "denied" | "reassigned" }`
- **Rate:** On-demand, only when `requesting_slot` is non-null in `/fleet/state_out`

### `GET /status`
- **Caller:** Fleet Networking (`net_node`)
- **Response:** `{ "slots": { "0": car_id|null, "1": car_id|null, "2": car_id|null } }`
- **Rate:** 1 Hz poll

### `POST /release`
- **Caller:** Fleet Networking (`net_node`)
- **Body:** `{ "car_id": str, "slot": int }`
- **Response:** `{ "status": "released" }`
- **Rate:** On-demand, called once when `/parking/state` reports `in_slot = 1`

---

## Track 1 (LiDAR, bonus) → Track 2: Perception → RL Agent

### `/scan`
- **Type:** `sensor_msgs/LaserScan`
- **Publisher:** Perception (`lidar_node`)
- **Subscriber:** RL Agent (`rl_agent_node`)
- **Rate:** 5.5 Hz (hardware limit of the RPLidar A1)
- **Fields that matter:** `ranges[]` array. The RL agent only needs the minimum value — treat anything below 0.15m as an immediate stop override, regardless of what the policy outputs.

---

## Track 2 → Track 3: RL Agent → ROS Integration

### `/cmd_vel`
- **Type:** `geometry_msgs/Twist`
- **Publisher:** RL Agent (`rl_agent_node`)
- **Subscriber:** ROS Integration (`motor_node`)
- **Rate:** 10 Hz
- **Fields that matter:**
  - `linear.x` — throttle, range -1.0 to 1.0
  - `angular.z` — steering, range -1.0 to 1.0
- **Notes:** This is the standard ROS2 velocity command message — using it means `motor_node` could in theory accept commands from any other ROS-compatible source (e.g. teleop) without modification.

---

## Track 3: ROS Integration (internal)

### `donkey_parking.launch.py`
Track 3 owns the launch file that brings up every node above in the correct order:
1. `camera_node`
2. `lidar_node` (optional, if hardware present)
3. `cv_state_node`
4. `net_node`
5. `rl_agent_node`
6. `motor_node`

Track 3 does not write logic for any of these nodes — their job is making sure the launch file starts them all, the topic names match this document exactly, and `ros2 topic list` shows everything connected as expected.

---

## Quick Reference Table

| Topic | Type | Publisher | Subscriber | Rate |
|---|---|---|---|---|
| `/camera/image_raw` | `sensor_msgs/Image` | Perception | Perception (internal) | 20 Hz |
| `/parking/state` | `std_msgs/Float32MultiArray` | Perception | RL Agent | 10 Hz |
| `/scan` | `sensor_msgs/LaserScan` | Perception | RL Agent | 5.5 Hz |
| `/fleet/state_in` | `std_msgs/String` | Fleet Networking | RL Agent | 5 Hz |
| `/fleet/state_out` | `std_msgs/String` | RL Agent | Fleet Networking | 5 Hz |
| `/cmd_vel` | `geometry_msgs/Twist` | RL Agent | ROS Integration | 10 Hz |

---

## Mock Nodes

To support continuous integration, a mock publisher exists for every topic above where a track might need to build before its upstream dependency is ready. Mocks live in `capstone/mocks/` and publish randomized but valid-shaped data at the correct rate. See `capstone/mocks/README.md` for usage.

---

*Last updated: [date] — changes require a PR and sign-off from any affected track.*
