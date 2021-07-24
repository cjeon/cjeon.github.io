---
layout: post
title: "Java+Kotlin Android project showing 0% kotlin coverage"
---

Today I spent 3+ hours digging this rather very simple issue and finally found a valid solution for this. I'm leaving notes here so that you don't spend yours on this.

## Problem description

Android test **surely** covers some .kt file, but Jacoco coverage report says non of .kt files are covered. Shows 0% coverage.

## Why it happened

Jacoco coverage report (.exec file) did contain coverage information for .kt classes, but the report generator did not find the sources of the classes (because it only considered java files) so it ignored .kt classes.

Android studio generates coverage report by internally using jacoco cli. This jacoco cli requires following arguments:

```cli
java -jar jacococli.jar report [<execfiles> ...] --classfiles <path> 
```

`execfiles` contain coverage information gathered from tests ran. `classfiles` are compiled `.class` files.

So what happened was, the Android studio only gave `.class` files of Java files and it did not pass `.class` files of the Kotlin files. That is why the final report did not contain kotlin classes. Knowing this, solution is easy.

## Solution

There are multiple solutions. First, we can manually create report with correct classfiles: both java and kotlin. You can download the jacoco executables from [here](https://www.jacoco.org/jacoco/).

Or we can keep using the Android studio's internal jacoco logic. Android studio passes `build/intermediates/javac` path to jacoco, and jacoco can only see .class files in that directory. 

So what we simply need to do is to move kotlin class files to this directory. Kotlin class files are located at `build/tmp/kotlin-classes`. Copy `.class` files from kotlin directory(`build/tmp/kotlin-classes`) to java directory(`build/intermediates/javac`), and generate the report again. You will see a beautiful report with precise coverage data.
