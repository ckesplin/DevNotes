```java
  @ElementCollection
  @CollectionTable(
    name="surveillance_plan_published_date",
    joinColumns=@JoinColumn(name="surveillance_plan_id")
  )
  @Column(name="published_date")
  @Builder.Default
  private List<Date> publishedDates = new ArrayList<>();
```