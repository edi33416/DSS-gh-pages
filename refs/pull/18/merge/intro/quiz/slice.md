---
nav_exclude: true
---
## Question Text

  {% assign q1_text = "Why, for the previous example, the lengths of a and b are different?" %}
  {% assign q1_choices = "`a` and `b` have the same lengths| `b` refers the same data as `a`, but it has no length constraints| a copy of `a` was saved into `b`, a dynamic array, and then the value 5 was appended to the dynamic array| operator `~` works for slices, but it has no impact on static arrays" | split: "| " %}
  {% assign q1_feedbacks = "`a` is a static array, so it's length is fixed, but we can still append data in that particular memory zone through a different reference| Correct!| `a` is a static array, so it's length is fixed, but we can still append data in that particular memory zone through a different reference| `a` is a static array, so it's length is fixed, but we can still append data in that particular memory zone through a different reference" | split: "| " %}
  {% assign q1_correct = 1 %}
  {% include mc-quiz.html text=q1_text choices=q1_choices answer=q1_correct feedback=q1_feedbacks %}
