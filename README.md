# git-bisect Demo

This project demos how to use [git-bisect](https://git-scm.com/docs/git-bisect).
The project contains a simple calculator that does not add correctly.

Your task is to find the commit that introduced the bug.

## What is bisecting?

Bisecting is the process of finding a commit that introduced a bug using in
binary search algorithm. The process is roughly as follows:

1. Define the range from the "last-known-good" to a broken commit.
1. Select a commit between the "last-known-good" and the earliest bad commit
   (usually by splitting the range in half).
1. Check if the commit contains the bug and report back to git.
1. Rinse and repeat step 2 and 3 until the bug-introducing commit is found.

The awkward part is to check whether the commit to check already introduced the
a bug. Luckily, this can be automated using a script you have in place ... which
you do, right?

## Usage

The repository contains two branches, `master` and `retrofit`. Both branches
have a broken calculator implemented as a simple shell script.

* The `master` branch includes a test for the calculator's `add` method,
  `adds-operands-test`. The test has been committed together with the `add` function
  (think TDD).

* The `retrofit` branch *initially* did not add a test for the broken
  calculator. This usually the case when developers don't subscribe to TDD or
  when the bug was not covered by the tests that were written by the time the
  `add` function was implemented. What you would usually do is to write a
  reproduction for the bug in the form of a test. For your convenience I've
  added the reproducing test in the most recent commit in `retrofit`,
  `adds-operands-reproduction-test`.

Each branch contains:

* A buggy calculator.
* A build script, `build`, that is used to build and test the calculator.
* A tag denoting the working revision: `good`.

### Bisecting in `master`

After cloning the repository start bisecting the `good..master` revision range.

```sh
$ git bisect start master good
```

You can test each commit manually while you are in bisect mode:

```sh
# You are in detached HEAD state at a commit between master and good.
$ ./build
# Check build results, if the build succeeded:
$ git bisect good
# otherwise:
$ git bisect bad
# Repeat until you found the breaking commit...

# When you are done, stop bisecting.
$ git bisect reset
```

You can also script the check by running `build` for every commit to be tested:

```sh
$ git bisect run ./build
```

Now watch git finding the commit that introduced the bug in the addition logic.

### Bisecting in `retrofit`

Bisecting without a proper reproduction that is already part of the commits to
be tested is a bit harder, but not impossible. You will have to apply the bug's
reproduction to each of the commits being tested.

In this example, we use [cherry-pick](https://git-scm.com/docs/git-cherry-pick)
to apply the reproduction to every commit in the `good..retrofit` revision
range. A little helper script, `start-bisect`, does the cherry-picking, build
script invocation and clean-up for us.

Switch to the `retrofit` branch and start bisecting the `bad..retrofit` revision
range.

```sh
# Create the retrofit branch from the remote retrofit branch.
$ git checkout -b retrofit origin/retrofit
# Start bisecting.
$ ./start-bisect
```
