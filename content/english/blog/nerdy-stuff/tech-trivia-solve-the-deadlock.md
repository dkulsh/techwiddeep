---
title: "Tech Trivia : Solve the Deadlock !"
meta_title: "Tech Trivia : Solve the Deadlock - Database Problem Solving Challenge"
description: "A piece of code gave a DB deadlock error. Figure out the reason! (and possibly the solution) - A hands-on problem-solving challenge with Python and database transactions."
date: 2021-09-22T00:00:00Z
image: "/images/nerdy-stuff/thread-deadlock-series/deadlock-error-screenshot.png"
categories: ["Technology", "Programming"]
author: "Deep Kulshreshtha"
tags: ["tech", "trivia", "deadlock", "database", "problem-solving", "python"]
draft: false
weight: 1
toc: true
---

A piece of code gave a DB deadlock error. Figure out the reason! (and possibly the solution)

*The code is in Python. But the trivia is less about the code and more about problem-solving.*

Write your answers in the comments section. The first few right answers get a shoutout.

Good luck!

To dive into the puzzle, let's examine the error details.

## Error

```python
deadlock detected DETAIL: Process 10137 waits for ShareLock on transaction 1842665572; blocked by process 32398. Process 32398 waits for ShareLock on transaction 1842665620; blocked by process 10137. HINT: See server log for query details. CONTEXT: while updating tuple (4368,11) in relation "person" while processing record
```

---

Now that we've seen the error, here's the relevant code snippet that triggered it.

## Code

```python
@transaction.atomic 
def update_person_parameters(self, person_id, **kwargs):

    Person = apps.get_model("params", "Person") 
    PersonSteps = apps.get_model("params", "PersonSteps") 
    PersonHeartRate = apps.get_model("params", "PersonHeartRate") 
    PersonTemperature = apps.get_model("params", "PersonTemperature") 
    TrainingSlot = apps.get_model("slot", "TrainingSlot") 
    TrainerBooking = apps.get_model("params", "TrainerBooking")

    steps = kwargs.pop("steps") 
    heart_rate = kwargs.pop("heart_rate") 
    temperature = kwargs.pop("temperature", [])

    trainer_bookings = self.model.objects.filter( 
        status__in=(self.model.STATUS_UPCOMING, self.model.COMPLETED, self.model.MISSED), 
        person__person_id=person_id, 
    ).values_list("primary_key", flat=True)

    if not trainer_bookings.exists(): 
        return # Nothing to do

    person_queryset = Person.objects.filter( 
        booking_id__in=trainer_bookings, 
        person_id=person_id 
    )

    person_queryset.update(**kwargs)

    centers = HealthCenter.objects\ 
        .filter(pk__in=person_queryset.values("trainer_bookings__centers__primary_key")[:1])

    mapping_queryset = TrainingSlot.objects\ 
        .filter(center__bookings=OuterRef("trainer_booking")) 
        .values("training_slot")[:1]

    Person.objects.update(session=Subquery(mapping_queryset))

    person_subquery_qs = Person.objects 
        .filter(trainer_booking=OuterRef("primary_key")) 
        .values("training_slot")[:1]

    TrainerBooking.objects.filter(guests__in=person_queryset)\ 
        .update(session=Subquery(person_subquery_qs))

    # A lot of other work 
    # ... 
    # ... 
    # ...
```

The solution is in the [next blog](/blog/nerdy-stuff/thread-deadlock-solved/).

## Navigation

**Next →**: [Thread Deadlock: Solved](/blog/nerdy-stuff/thread-deadlock-solved/)
