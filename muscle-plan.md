# Muscle Points Bar Chart Implementation Plan

## Overview

Implement a fullstack muscle points bar chart on the statistics page. The chart shows per-muscle involvement points calculated from workout activity sets (primary muscle = 1 point, secondary = 0.5 points per set), displayed as a Recharts `BarChart` with multi-color bars, percentage tooltips, and a hardcoded toggle for single-color mode.

## Current State Analysis

The codebase is largely pre-wired for this feature:

- **Backend**: Route (`POST /api/trainee-analytics/muscle-volume/:userId`), controller handler, and service method all exist, but the service returns `[]` (TODO stub).
- **Frontend**: API function, TanStack Query hook, and route-level data fetching all exist. The dashboard has a commented-out placeholder.
- **Shared**: `MuscleInvolvementPointsEnum` (PRIMARY=1, SECONDARY=0.5) and `calculateMusclePointsForExercise` exist. The `MuscleGroupVolumeData` type exists but has volume-specific fields that need replacing.

### Key Discoveries:

- Controller at `traineeAnalyticsController.ts:6-44` does NOT pass `req` to the service, unlike all other analytics service methods. This means no `authorizeDataAccess()` call. Must fix.
- `ExerciseMuscleType` model uses `exercise_muscles_type` table with `exercise_id`, `muscle_id`, `type` (PRIMARY/SECONDARY)
- `WorkoutActivity` model has `exercise_id`, `skipped` boolean, `workout_instance_id`
- Associations: Exercise belongsToMany Muscle through `exercise_muscles_type` (`models/index.ts:91-104`)
- WeightChart uses `ResponsiveContainer width="100%" height={300}`, CSS variable colors, custom tooltip, `useTranslations` for i18n

## Desired End State

A horizontal bar chart appears on the statistics/analytics dashboard showing each muscle's accumulated points. Bars are sorted descending by points. On hover, a tooltip shows the muscle name, point total, and percentage of all points. A hardcoded boolean in the component toggles between multi-color and single `bg-primary` color mode.

### Verification:

1. Select a trainee with completed workouts on the statistics page
2. The muscle points chart renders below the frequency chart
3. Bars are sorted by highest points first
4. Hovering a bar shows: muscle name, points value, and percentage (e.g., "75%")
5. The multi-color mode shows each bar in a different color from the palette
6. Toggling the boolean to `false` renders all bars in `bg-primary` color

## What We're NOT Doing

- No new database migration (existing tables are sufficient)
- No new API endpoint (reusing existing route/controller/hook)
- No new query hook or API function (already wired)
- No filtering by muscle group or exercise type
- No animation/transitions beyond Recharts defaults
- No separate "volume" chart (we're repurposing the type for points)

## Implementation Approach

Since the entire API pipeline (route -> controller -> hook) already exists but the service returns `[]`, this is primarily:

1. Replace the shared type to match muscle points data
2. Fix the service signature and implement the Sequelize query
3. Create the chart component
4. Uncomment/wire it in the dashboard

---

## Phase 1: Shared Types & Error Code

### Overview

Replace the `MuscleGroupVolumeData` interface with muscle-points-specific fields, and add a missing error code for the service.

### Changes Required:

#### 1. Replace `MuscleGroupVolumeData` type

**File**: `packages/shared-my-training-app/src/types/index.ts`
**Lines**: 280-290

Replace the current interface:

```typescript
export interface MuscleGroupVolumeData {
  muscleId: number;
  englishName: string;
  hebrewName: string;
  involvement: MuscleInvolvementEnum;
  totalVolume: number;
  sessionCount: number;
  averageVolumePerSession: number;
  volumePercentage: number;
  exerciseCount: number;
}
```

With:

```typescript
export interface MuscleGroupVolumeData {
  muscleId: number;
  englishName: string;
  hebrewName: string;
  points: number;
}
```

This keeps the same type name so all imports across the codebase remain valid.

#### 2. Add error code to shared error map interface

**File**: `packages/shared-my-training-app/src/codesMap/errorMap.ts`
**Location**: After line 153 (`FAILED_PERFORMANCE_SUMMARY`)

Add:

```typescript
FAILED_MUSCLE_GROUP_VOLUME_DISTRIBUTION: () => ErrorResponse;
```

#### 3. Add error code to backend error map

**File**: `packages/backend-my-training-app/src/backend/codesMap/errorMap.ts`
**Location**: After `FAILED_PERFORMANCE_SUMMARY` entry (line ~435)

Add:

```typescript
FAILED_MUSCLE_GROUP_VOLUME_DISTRIBUTION: () => ({
  ERROR_CODE: "FAILED_MUSCLE_GROUP_VOLUME_DISTRIBUTION",
  message: "Failed to get muscle group volume distribution",
}),
```

### Success Criteria:

#### Automated Verification:

- [x] Shared package builds: `cd packages/shared-my-training-app && pnpm build`
- [ ] Backend typechecks (no unused fields referenced)
- [ ] Frontend typechecks (no unused fields referenced)

---

## Phase 2: Backend Service Implementation

### Overview

Fix the controller to pass `req` to the service, implement the service query to aggregate muscle points from workout activities.

### Changes Required:

#### 1. Fix controller to pass `req`

**File**: `packages/backend-my-training-app/src/controllers/traineeAnalyticsController.ts`
**Lines**: 29-32

Change from:

```typescript
const volumeDistribution =
  await traineeAnalyticsService.getMuscleGroupVolumeDistribution(
    userId,
    dateRange,
  );
```

To:

```typescript
const volumeDistribution =
  await traineeAnalyticsService.getMuscleGroupVolumeDistribution(
    req,
    userId,
    dateRange,
  );
```

#### 2. Implement the service method

**File**: `packages/backend-my-training-app/src/backend/services/traineeAnalyticsService.ts`
**Lines**: 46-52

Replace the TODO stub with the full implementation. The method needs to:

1. Accept `req` parameter for authorization
2. Call `authorizeDataAccess(req, userId)`
3. Query `WorkoutActivity` for the user in the date range:
   - Join to `WorkoutInstance` (filter by `user_id`, date range, `COMPLETED` status)
   - Join to `Exercise` -> `muscles` (through `exercise_muscles_type`)
   - Filter `skipped: false`
4. Aggregate points per muscle:
   - For each activity (set), look up its exercise's muscles
   - PRIMARY muscle = +1 point, SECONDARY muscle = +0.5 point
   - Sum across all sets per muscle
5. Sort descending by points
6. Return `MuscleGroupVolumeData[]`

Add required imports:

```typescript
import {
  Exercise,
  Muscle,
  ExerciseMuscleType,
} from "../sequelize/models/index.js";
import { MuscleInvolvementPointsEnum } from "@guy-vaserman/shared-my-training-app";
```

Implementation:

```typescript
export async function getMuscleGroupVolumeDistribution(
  req: Request,
  userId: number,
  dateRange: DateRange,
): Promise<MuscleGroupVolumeData[]> {
  await authorizeDataAccess(req, userId);
  try {
    // Query all non-skipped activities for the user in date range
    const activities = await WorkoutActivity.findAll({
      where: {
        skipped: false,
      },
      include: [
        {
          model: WorkoutInstance,
          as: "workout_instance",
          where: {
            user_id: userId,
            start_time: {
              [Op.between]: [dateRange.startDate, dateRange.endDate],
            },
            workout_status: WorkoutStatus.COMPLETED,
          },
          attributes: [],
        },
        {
          model: Exercise,
          as: "exercise",
          attributes: ["exercise_id"],
          include: [
            {
              model: Muscle,
              as: "muscles",
              attributes: ["muscle_id", "english_name", "hebrew_name"],
              through: {
                attributes: ["type"],
              },
            },
          ],
        },
      ],
      attributes: ["activity_id", "exercise_id"],
    });

    // Aggregate points per muscle
    const musclePointsMap = new Map<
      number,
      { englishName: string; hebrewName: string; points: number }
    >();

    for (const activity of activities) {
      const activityJson = activity.toJSON() as any;
      const muscles = activityJson.exercise?.muscles;
      if (!muscles) continue;

      for (const muscle of muscles) {
        const muscleId = muscle.muscle_id;
        const involvementType = muscle.exercise_muscles_type?.type;
        if (!involvementType) continue;

        const points =
          MuscleInvolvementPointsEnum[
            involvementType as keyof typeof MuscleInvolvementPointsEnum
          ] ?? 0;

        const existing = musclePointsMap.get(muscleId);
        if (existing) {
          existing.points += points;
        } else {
          musclePointsMap.set(muscleId, {
            englishName: muscle.english_name,
            hebrewName: muscle.hebrew_name,
            points,
          });
        }
      }
    }

    // Convert to array and sort descending by points
    const result: MuscleGroupVolumeData[] = Array.from(
      musclePointsMap.entries(),
    )
      .map(([muscleId, data]) => ({
        muscleId,
        englishName: data.englishName,
        hebrewName: data.hebrewName,
        points: data.points,
      }))
      .sort((a, b) => b.points - a.points);

    return result;
  } catch (error) {
    throw BACKEND_ERROR_MAP.FAILED_MUSCLE_GROUP_VOLUME_DISTRIBUTION();
  }
}
```

Note: The `Exercise` and `Muscle` models need to be imported. `Exercise` is already imported at line 13. `Muscle` and `ExerciseMuscleType` need to be added to the import from `../sequelize/models/index.js`. Also add `MuscleInvolvementPointsEnum` to the shared import.

### Success Criteria:

#### Automated Verification:

- [x] Backend builds: `cd packages/backend-my-training-app && pnpm build`
- [ ] Backend starts without errors
- [ ] Existing analytics tests still pass (if any)

#### Manual Verification:

- [ ] Call `POST /api/trainee-analytics/muscle-volume/:userId` with a valid date range and user with completed workouts
- [ ] Response contains array of `{ muscleId, englishName, hebrewName, points }` sorted by points descending
- [ ] Skipped sets are excluded from the calculation
- [ ] Unauthorized access returns appropriate error

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the API returns correct data before proceeding to the next phase.

---

## Phase 3: Frontend MusclePointsChart Component

### Overview

Create a new Recharts `BarChart` component with multi-color bars, percentage tooltips, and a hardcoded toggle for single-color mode.

### Changes Required:

#### 1. Create MusclePointsChart component

**File**: `packages/frontend-my-training-app/src/components/analytics/charts/MusclePointsChart.tsx` (NEW)

Key design decisions:

- Follows `WeightChart` pattern: `ResponsiveContainer width="100%" height={300}`, card wrapper, custom tooltip
- Uses `BarChart` + `Bar` from Recharts (already available, same package)
- Color palette: array of 12+ distinct HSL colors, assigned by bar index
- Hardcoded `USE_MULTI_COLOR = true` boolean at top of file
- Tooltip shows: muscle name, points, percentage of total
- X-axis: muscle names (locale-aware using `englishName`/`hebrewName`)
- Y-axis: points
- Data arrives pre-sorted from backend (descending by points)

```typescript
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  Cell,
} from "recharts";
import { useTranslations } from "@/hooks/useTranslations";
import { useLocale } from "@/hooks/use-locale";
import type { MuscleGroupVolumeData } from "@guy-vaserman/shared-my-training-app";
import type { TooltipProps } from "recharts";

// Toggle between multi-color and single primary color
const USE_MULTI_COLOR = true;

const COLORS = [
  "hsl(210, 80%, 55%)",  // blue
  "hsl(340, 75%, 55%)",  // pink
  "hsl(150, 65%, 45%)",  // green
  "hsl(35, 90%, 55%)",   // orange
  "hsl(270, 65%, 60%)",  // purple
  "hsl(185, 70%, 45%)",  // teal
  "hsl(0, 75%, 55%)",    // red
  "hsl(55, 80%, 50%)",   // yellow
  "hsl(300, 50%, 55%)",  // magenta
  "hsl(170, 60%, 40%)",  // dark teal
  "hsl(25, 85%, 55%)",   // dark orange
  "hsl(230, 60%, 60%)",  // indigo
];

interface MusclePointsChartProps {
  data: MuscleGroupVolumeData[];
  className?: string;
}

export default function MusclePointsChart({
  data,
  className,
}: MusclePointsChartProps) {
  const t = useTranslations(
    "Components.TraineeAnalyticsDashboard.musclePointsChart",
  );
  const locale = useLocale();
  const isHebrew = locale === "he";

  const totalPoints = data.reduce((sum, d) => sum + d.points, 0);

  const chartData = data.map((d) => ({
    name: isHebrew ? d.hebrewName : d.englishName,
    points: d.points,
    percentage:
      totalPoints > 0 ? Math.round((d.points / totalPoints) * 100) : 0,
  }));

  const CustomTooltip = ({
    active,
    payload,
  }: TooltipProps<number, string>) => {
    if (active && payload && payload.length) {
      const entry = payload[0].payload;
      return (
        <div className="bg-background border rounded-lg p-3 shadow-lg">
          <p className="font-medium">{entry.name}</p>
          <p className="text-primary">
            {t("points")}: {entry.points}
          </p>
          <p className="text-muted-foreground text-sm">
            {entry.percentage}%
          </p>
        </div>
      );
    }
    return null;
  };

  return (
    <div className={className}>
      <div className="mb-4">
        <h3 className="text-lg font-semibold">{t("title")}</h3>
        <p className="text-sm text-muted-foreground">{t("description")}</p>
      </div>
      <ResponsiveContainer width="100%" height={300}>
        <BarChart
          data={chartData}
          margin={{ top: 5, right: 30, left: 20, bottom: 5 }}
        >
          <CartesianGrid strokeDasharray="3 3" className="opacity-30" />
          <XAxis
            dataKey="name"
            fontSize={11}
            tick={{ fill: "hsl(var(--foreground))" }}
            interval={0}
            angle={-45}
            textAnchor="end"
            height={80}
          />
          <YAxis
            fontSize={12}
            tick={{ fill: "hsl(var(--foreground))" }}
          />
          <Tooltip content={<CustomTooltip />} />
          <Bar dataKey="points" radius={[4, 4, 0, 0]}>
            {chartData.map((_, index) => (
              <Cell
                key={`cell-${index}`}
                fill={
                  USE_MULTI_COLOR
                    ? COLORS[index % COLORS.length]
                    : "hsl(var(--primary))"
                }
              />
            ))}
          </Bar>
        </BarChart>
      </ResponsiveContainer>
    </div>
  );
}
```

#### 2. Add i18n keys

**File**: `packages/frontend-my-training-app/src/i18n/dictionary/en.json`

Add under `Components.TraineeAnalyticsDashboard`:

```json
"musclePointsChart": {
  "title": "Muscle Points",
  "description": "Muscle involvement points from your workouts",
  "points": "Points"
}
```

**File**: `packages/frontend-my-training-app/src/i18n/dictionary/he.json`

Add under `Components.TraineeAnalyticsDashboard`:

```json
"musclePointsChart": {
  "title": "MISSING-KEY-1",
  "description": "MISSING-KEY-2",
  "points": "MISSING-KEY-3"
}
```

Then run `pnpm intl-sync` to sync across all dictionaries.

### Success Criteria:

#### Automated Verification:

- [ ] Frontend builds: `cd packages/frontend-my-training-app && APP_STAGE=dev pnpm build`
- [ ] No TypeScript errors
- [ ] i18n sync passes: `cd packages/frontend-my-training-app && pnpm intl-sync`

---

## Phase 4: Dashboard Integration

### Overview

Wire `MusclePointsChart` into `TraineeAnalyticsDashboard` and ensure the `muscleGroupVolume` prop is passed through to the chart.

### Changes Required:

#### 1. Add chart to dashboard

**File**: `packages/frontend-my-training-app/src/components/analytics/TraineeAnalyticsDashboard.tsx`

Import and render the chart. Replace the commented-out section (lines 79-88) with:

```typescript
import MusclePointsChart from "./charts/MusclePointsChart";
```

And in the JSX, replace the commented-out block with:

```tsx
{
  muscleGroupVolume && muscleGroupVolume.length > 0 && (
    <MusclePointsChart
      data={muscleGroupVolume}
      className="bg-card border rounded-lg p-4 lg:p-6"
    />
  );
}
```

Also destructure `muscleGroupVolume` from the props (it's currently received but not used).

#### 2. Verify data flow

The data flow is already complete:

- `statistics.tsx` route: calls `useGetMuscleGroupVolumeDistribution()` at line 91
- Passes `muscleGroupVolumeQuery.data` to `StatisticsPage` as `muscleGroupVolume` at line 171
- `StatisticsPage` passes it to `TraineeAnalyticsDashboard` at line 335
- `TraineeAnalyticsDashboard` receives it as `muscleGroupVolume` prop (line 20) -- just needs to render it

### Success Criteria:

#### Automated Verification:

- [ ] Frontend builds: `cd packages/frontend-my-training-app && APP_STAGE=dev pnpm build`
- [ ] No unused variable warnings for `muscleGroupVolume`
- [ ] No TypeScript errors

#### Manual Verification:

- [ ] Navigate to statistics page with a trainee that has completed workouts
- [ ] Muscle points bar chart renders below the frequency chart
- [ ] Bars are sorted descending (highest points first)
- [ ] Hovering a bar shows: muscle name, points, percentage
- [ ] Multi-color mode works (each bar different color)
- [ ] Setting `USE_MULTI_COLOR = false` renders all bars in primary color
- [ ] Chart renders correctly on mobile (responsive)
- [ ] Hebrew locale shows Hebrew muscle names
- [ ] Works in both coach statistics page and trainee dashboard embedded view

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the chart looks correct and the UX is acceptable.

---

## Testing Strategy

### Unit Tests:

- Test the muscle points aggregation logic (if extracted to a pure function in calculations lib)
- Test edge cases: user with no workouts returns `[]`, exercises with no muscles return `[]`

### Integration Tests:

- Backend: call `getMuscleGroupVolumeDistribution` with a seeded user and verify the response shape
- Verify skipped sets are excluded
- Verify authorization denies access for unauthorized users

### Manual Testing Steps:

1. Select a trainee with varied workout history (multiple exercises, primary + secondary muscles)
2. Verify point totals match manual calculation
3. Change date range and confirm chart updates
4. Test with a trainee with no workouts (chart should not render)
5. Test mobile viewport for responsive layout
6. Switch to Hebrew and verify muscle names

## Performance Considerations

- The query joins WorkoutActivity -> WorkoutInstance -> Exercise -> Muscle which could be slow for users with thousands of sets. If this becomes an issue, consider a raw SQL query with GROUP BY instead of application-level aggregation.
- The current approach fetches all activities and aggregates in JS. For a first implementation this is acceptable and matches the pattern of other analytics methods in the codebase.

## References

- Structured spec: `thoughts/structured/muscle-points-bar-chart.md`
- Raw requirements: `thoughts/raw/muscle.md`
- Research: `thoughts/shared/research/2026-01-27-muscle-points-bar-chart.md`
- Backend service: `packages/backend-my-training-app/src/backend/services/traineeAnalyticsService.ts:46-52`
- Frontend dashboard: `packages/frontend-my-training-app/src/components/analytics/TraineeAnalyticsDashboard.tsx`
- WeightChart pattern: `packages/frontend-my-training-app/src/components/analytics/charts/WeightChart.tsx`
- Shared types: `packages/shared-my-training-app/src/types/index.ts:280-290`
- Shared calculation: `packages/shared-my-training-app/src/calculations/index.ts:11`
