## Data Refactoring Advisor Tool

A tool that enables developers to create graphs based on database workflow captured in a SQL Tuning Set.  One the graph is created, community detection is performed to find tables that are closely related.  These communities provide an abstraction for building micro services on top of an existing database.  The communities are analagous to a service's bounded context.

The tool is divided into 2 components

1. DRA Backend - A Spring Boot servcie that provides property graph support

2. DRA Frontend - A React App for 

    1. Creating SQL Tuning Sets
    2. Rendering Graphs
    3. Rendering Community Detection
