I want to add a new fullstack feature
THe feautre will be to add a new bar chart to the statstics page

It will be a calcualtion that I will now give you

The Y is points
The X acess will be each name of each muscle involved and they will be sorted by the most points and acesseding
This calcaulation is a feature of statstics, and like the other ones it works by calcaualtioning as the smallest unit of a workout will be the workoutactivity which is a set, so basically we need to query all of those and then do the calcualtion on it

So we have currently we have workout activities which has a exercise_id which exercise_id are mapped to muscles in hte
src/backend/sequelize/models/ExerciseMuscleType.ts

we have primary or secoundry, so in our calcualtion our primary is worth 1 point and secoundry 0.5 points

TYPESCRIPT
example:

## Mock Data

```
exerciseMuscles = [
  { exercise_id: 1, muscle: "Chest",   type: "primary" },
  { exercise_id: 1, muscle: "Triceps", type: "secondary" },
]

workoutActivities = [
  { exercise_id: 1 },  // set 1
  { exercise_id: 1 },  // set 2
]
```

## Function (pseudo)

```
for each activity in workoutActivities:
  muscles = getMusclesForExercise(activity.exercise_id)
  for each muscle:
    points += (type == "primary") ? 1 : 0.5

return sortByPointsDescending(musclePoints)
```

## Result

```
[
  { muscle: "Chest",   points: 2 },   // 2 sets × 1 (primary)
  { muscle: "Triceps", points: 1 },   // 2 sets × 0.5 (secondary)
]
```

STLYING WISE IT WILL TAKE as much space as weightChart and distrubtuion , make MOBILE RESPONSIVE NESS A good UX

Each bar will have a differnet coolor just make maybe a map of colors via index

Important to SORT them by biggest

I want also an option to instead of having it differnt colors i WANT a frontend boolean that i can change true false if we are using our bg-primary color

VERY IMPORANT WE NEED TO ADD suppport for when we hover to use % so if i did 3 point for back and 1 point for leg so then 75% bck and 25% leg
