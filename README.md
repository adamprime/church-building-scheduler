# Church Building Scheduler

A single-file HTML tool for scheduling multiple units (wards/branches) in a shared church building. Drag-and-drop interface with real-time constraint validation.

## Features

- **Visual Timeline**: Drag blocks to adjust meeting times
- **Click to Flip**: Click any block to swap SAC/2ND hour order
- **Real-time Validation**: Instant feedback on scheduling conflicts
- **Shareable Links**: URL updates as you adjust—share or bookmark any configuration
- **Dark Mode**: Toggle for comfortable viewing
- **Print/PDF**: Export schedules for distribution

## Usage

1. Open `scheduler.html` in any modern web browser
2. Drag unit blocks left/right to adjust start times
3. Click a block to flip the order (SAC first vs 2ND hour first)
4. Watch the constraint panel for conflicts
5. Use "Copy Link" to share your schedule configuration

## Customizing for Your Building

The scheduler is designed to be adapted for different buildings with different units and constraints. All configuration is contained within `scheduler.html`.

### 1. Changing Units

Find the `UNITS` array near the top of the `<script>` section (~line 630):

```javascript
const UNITS = [
    { id: 'liberty', name: 'Liberty', color: '#2563eb' },
    { id: 'rushcreek', name: 'Rush Creek', color: '#059669' },
    { id: 'sanrafael', name: 'San Rafael', color: '#d97706' },
    { id: 'ysa', name: 'YSA', color: '#7c3aed' }
];
```

Update the `id` (used internally), `name` (displayed), and `color` for your units.

### 2. Setting Default Schedule

Find the `DEFAULT_STATE` object (~line 638):

```javascript
const DEFAULT_STATE = {
    liberty: { start: 9, sacFirst: true },      // 9:00 AM, Sacrament first
    rushcreek: { start: 11.5, sacFirst: true }, // 11:30 AM, Sacrament first
    sanrafael: { start: 11, sacFirst: false },  // 11:00 AM, 2ND hour first
    ysa: { start: 14, sacFirst: false }         // 2:00 PM, 2ND hour first
};
```

- `start`: Time in 24-hour decimal format (9 = 9:00 AM, 11.5 = 11:30 AM, 14 = 2:00 PM)
- `sacFirst`: `true` = Sacrament then 2ND hour; `false` = 2ND hour then Sacrament

### 3. Modifying Constraints

Constraints are checked in the `checkConstraints()` function (~line 920). The function builds a list of pass/fail/warn results and calculates a score.

#### Types of Constraints

**Hard constraints** (must not violate):
- Return `status: 'fail'` and subtract significant points (e.g., 25-30)

**Soft constraints** (preferred):
- Return `status: 'warn'` and subtract fewer points (e.g., 5)

#### Common Constraint Patterns

**No overlap between two time periods:**
```javascript
if (timesOverlap(timeA, timeB)) {
    constraints.push({
        text: 'Unit A & Unit B conflict',
        status: 'fail'
    });
    score -= 25;
}
```

**Minimum gap between meetings:**
```javascript
const gap = getGapMinutes(earlierTime, laterTime);
if (gap < 15) {
    constraints.push({ text: 'Not enough gap', status: 'fail' });
    score -= 25;
} else if (gap < 30) {
    constraints.push({ text: 'Gap is tight', status: 'warn' });
    score -= 5;
}
```

#### Helper Functions

- `getSacTime(unitId)` - Returns `{ start, end }` for a unit's sacrament meeting
- `get2ndTime(unitId)` - Returns `{ start, end }` for a unit's second hour
- `getBlockTime(unitId)` - Returns `{ start, end }` for the entire 2-hour block
- `timesOverlap(a, b)` - Returns `true` if two time periods overlap
- `getGapMinutes(earlier, later)` - Returns minutes between two time periods

### 4. Adjusting Time Window

Change the visible timeline range (~line 626):

```javascript
const START_HOUR = 8.5;  // 8:30 AM
const END_HOUR = 17;     // 5:00 PM
```

### 5. Updating the Reference Panel

Don't forget to update the "Scheduling Constraints Reference" section in the HTML (~line 545) to document your building's rules for users.

## Example: Current Constraints

This scheduler is configured for the Clayview Building with these rules:

- **Chapel**: Only one sacrament meeting at a time; 15 min minimum between (30 min preferred)
- **Parking**: Liberty & Rush Creek blocks cannot overlap (30 min gap required)
- **Classrooms**: Liberty, Rush Creek, and San Rafael cannot have overlapping 2ND hours (YSA can overlap with anyone)
- **Time Window**: Prefer 9:00 AM - 4:00 PM

## License

MIT - Feel free to adapt for your building's needs.
