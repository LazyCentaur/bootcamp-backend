# 00-entrega-modelado

## Basic case

**eLearning** portal with courses of **programing**

### Overview

![image](./content/general.png)


### 1. Home

**SUBSET PATTERN** - Unused data is extracted in the home page to optimize space and optimize space in the Working Set.

![image](./content/home.png)

The home page has been embedded in a table with the aim of loading as quickly as possible, established in the Workset, it will not take up much space since we only have the last_courses and popular_courses objects with the most recently added courses, their names and their categories to be able to classify them on the home page. We will maintain a relationship with the **course** table through the course id.

Duplicate data should not change, since once a course is uploaded, it would be exceptional for the name to change. 

### 2. Courses

![image](./content/course.png)

**EXTENDED REF pattern** - When we access the details of the course, we want to load the page quickly, as we have a maximum of 20 videos per course, we decide to embed the lessons within the course, the video and the description will be links so they will not take up much space.

This would also be in the Working Set.

### 3. Authores

They are decided to be embedded within the courses, since it is information that should not change, it can be duplicated without problem.

### 3. Biography

![image](./content/biography.png)

**SUBSET pattern** - An author's biography is something that is not going to be visited much, it can be displayed in another separate table without problem, and **not** have it in the Working Set

## Relaciones

### Categories

- A category can have several last_courses.
- A category can have several popular_courses.
- A course can only appear once in last_courses.
- A course can only appear once in popular_courses.
- A course can only have one category.
- A lesson can have several categories.
- An author can only have one biography.
- A biography can have several lessons.









