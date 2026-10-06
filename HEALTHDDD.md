
## 1. Domain

Imagine a university enrollment system.

A student wants to enroll in courses, but the university has rules such as:

A student cannot enroll in more than 6 courses.
A student cannot enroll in the same course twice.
A course must have available capacity.
A student must meet the course prerequisites.
An enrollment must belong to a specific student.

## 2. Aggregate Root: CourseEnrollment

The aggregate root is the main object responsible for controlling the enrollment information

CourseEnrollment
│
├── StudentId
├── Semester
├── Status
│
└── EnrolledCourses
       ├── Course A
       ├── Course B
       └── Course C


## 3. Example Invariants

Let's assume the university has these limits:

Rule	Limit
Maximum courses per semester	6
Maximum students in a course	30
Duplicate enrollment	Not allowed
Prerequisites	Must be satisfied

## Example

If a student already has:
1. Database Systems
2. Java Programming
3. Computer Networks
4. Mathematics
5. Operating Systems
6. Software Engineering

 ## They already have 6 courses.

Trying to add another course should fail

CourseEnrollment
       |
       | Enroll(AI)
       ↓
Check maximum = 6
       |
       ↓
Already 6 courses
       |
       ↓
REJECT

## 4. Domain Events

When something important happens in the domain, the aggregate can produce a domain event.

For example
CourseEnrollmentCreated
CourseAdded
CourseEnrollmentRejected
CourseDropped



## "Something important has happened in the business."

For example

Student enrolls in AI
        ↓
CourseEnrollment checks rules
        ↓
Rules pass
        ↓
Course is added
        ↓
CourseEnrolled event is emitted

## 5. Simple DDD Model

┌─────────────────────────────────────┐
│       CourseEnrollment              │
│       <<Aggregate Root>>            │
│                                     │
│  StudentId                          │
│  Semester                           │
│  Courses                            │
│  Status                             │
│                                     │
│  + enrollCourse()                   │
│  + dropCourse()                     │
│  + canEnroll()                      │
└─────────────────┬───────────────────┘
                  │
                  │ emits
                  ↓
        ┌─────────────────────┐
        │ CourseEnrolled      │
        │ Domain Event        │
        └─────────────────────┘

 ## 6. Example Pseudocode

     class CourseEnrollment {

    private StudentId studentId;
    private Semester semester;
    private List<Course> courses;

    private static final int MAX_COURSES = 6;

    public void enrollCourse(Course course) {

        // Invariant 1: Maximum course limit
        if (courses.size() >= MAX_COURSES) {
            throw new EnrollmentLimitExceededException();
        }

        // Invariant 2: No duplicate course
        if (courses.contains(course)) {
            throw new CourseAlreadyEnrolledException();
        }

        // Invariant 3: Check prerequisites
        if (!course.prerequisitesSatisfiedBy(studentId)) {
            throw new PrerequisiteNotSatisfiedException();
        }

        // Add course
        courses.add(course);

        // Emit domain event
        raiseEvent(
            new CourseEnrolled(
                studentId,
                course.getCourseId(),
                semester
            )
        );
    }
}

## 7. What happens when a student enrolls?

Student requests enrollment
          ↓
CourseEnrollment
          ↓
Check maximum courses
          ↓
Check duplicate
          ↓
Check prerequisites
          ↓
All rules pass
          ↓
Add course
          ↓
Emit CourseEnrolled event

Failed enrollment

Student requests enrollment
          ↓
CourseEnrollment
          ↓
Student already has 6 courses
          ↓
Invariant violated
          ↓
Reject enrollment
          ↓
No CourseEnrolled event


    
