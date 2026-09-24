# Module 02 — Extra Exercises

## Exercise 1 — Least-privilege sudo

Set up a user who can only restart one specific service via sudo, nothing else. Verify by trying (and failing) to run a different privileged command as that user.

## Exercise 2 — Group-based access control (the alice/bob/tom test)

Create three users: `alice`, `bob`, and `tom`. Set up
`/home/shared` such that `alice` and `bob` can create and access
files there — sharing them with each other automatically — while
`tom` can't access it at all, and *without* either user manually
`chmod`-ing each new file after creating it.

The real test: `alice` creates a file in `/home/shared`, and `bob`
can immediately read/edit it with no extra step from `alice`, while
`tom` gets permission denied. (Hint: this is module 04's setgid
trick, module 05's `umask`, and group membership from this module,
combined — get the directory's permissions and group ownership
right once, and it should just work for every file created inside it
afterward.)

Show the directory's permissions and the success/failure output for
each user as proof.

## Exercise 3 — Audit your own sudoers

Run `sudo cat /etc/sudoers` and `sudo ls /etc/sudoers.d/` on your VM and explain, in your own words, every line you find.

