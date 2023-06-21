# Script
Today we're going to be talking about rewriting Git history. There's probably some stuff in here that you already know but hopefully there's some new stuff that you don't. 

If you're a Git expert and know everything, well we're going to be having a history lesson too. B/c you know who _loved_ to rewrite history? Henry VIII

So we're going to be using his family tree and his marriages to talk about managing your Git history.

I have a repo set up with an initial commit to start learning about the House of Tudor.

The House of Tudor started with Henry VII. If you've ever heard of the War of the Roses or the Lancasters and Yorks, that's this guy.

After he won the crown, he married Elizabeth of York and they went on to have 4 kids.

For this talk, we're just going to focus on the line of succession or who gets to wear the crown. 

## Arthur, set up cherry-pick

Arthur was the first born boy and the heir apparent. So we're going to make a new branch to talk about Arthur's rise to the throne.

```
git checkout -b arthur
```

When Arthur was a child, Henry VII planned his engagement to Catherine of Aragon to cement an alliance with Spain.

They were married when he was 16.
  
Add `A[Arthur]---C[Catherine of Aragon]`

```
git commit -m 'Married Catherine of Aragon'
```

6 months later, he died and Henry became the heir.

```
git checkout main
```

## Catherine of Aragon, cherry-pick

Henry became king 7 years later when his father died.

```
git checkout -b henry
```

Add `H8[Henry VIII]`

```
git add .
git commit -m 'Henry becomes king'
```
Shortly afterward, he married Catherine of Aragon to keep that alliance with Spain. 

Git cherry-pick lets you grab commits from other branches and then apply them in your branch as a new commit.

We already made an update almost the same as this one, so I'm going to cherry-pick that into this branch

I use git log --oneline to easily see my SHAs

```
git checkout arthur
git log --oneline
git checkout henry
git cherry-pick [SHA for 'Married Catherine of Aragon']
```

If there are no conflicts, it will apply the commit as-is with a new SHA. But we have conflicts so I'm going to resolve those.

Edit the commit
  
```
git add .
git cherry-pick --continue
```

## Mary, git add -p
7 years into marriage, Catherine gave birth to a baby girl named Mary

Add to the top of the flowchart:
```
subgraph Gen3
  M[Mary]
end
```

Add to the bottom of the flowchart:
```
Gen2---Gen3
```

Git has a really powerful little flag that you can use on a lot of its commands called "patch" or "-p"

The patch flag lets you complete a command in chunks. So in this case we can add our changes in chunks.

```
git add -p 
# yes to both
git reset .
```
There are a lot of different commands while you are patching. If you want to look at the options that question mark will show you what they all mean.

```
git add -p
# show how to edit (don't include Mary but everything else)
git commit -m 'Setup 3rd generation'
git add -p
git commit -m 'Mary was born'
```

## Anne Boleyn, git stash
After more than 20 years, Henry started an affair with Anne Boleyn. 

While he attempted to get the church to grant him an annullment from his marriage with Catherine, he secretly married Anne Boleyn.

Add new line `H8---A1[Anne Boleyn]`

But this had to remain a secret b/c at this point, the Vatican was still the Church in England and Henry was technically married to two people.

So I'm going to stash these changes while we keep them secret

```
git stash
git stash list
git stash pop
```

There are some fun options that you can send to stash: -p, -k, and -u
-p lets you stash in chunks, -k lets you stash put keep the things you put in the staging area, and -u allows you to stash untracked files

```
git stash save -k
git stash save -u
```

I like to use git stash save with those options so that I can assign a little message to keep track of things in my stash list

```
git stash save 'Married Anne Boleyn'
git stash list
```

## Catherine of Aragon divorce, git revert
A few months after his secret wedding to Anne, Henry managed to get his marriage to Catherine declared null & void.

There are a few ways to get rid of commits that you've already made. The first way that we're going to see is the one most people are familiar with: revert.

A revert creates another commit to undo a previous one.

```
git log --oneline
git revert [SHA of 'Married Catherine of Aragon']
```

Henry had another small wedding ceremony to Anne Boleyn now that he was legally not married.

```
git stash pop
git commit -m 'Married Anne Boleyn'
```

## Elizabeth, interactive rebase
Shortly after, Anne Boleyn gave birth to Elizabeth

Add `E1[Elizabeth]`
  
```
git commit -m "Elizabeth was born"
```

With pressure from Anne who wanted her daughter to be next in line to throne, Henry updated his Acts of Succession to make Mary illegitimate 

Now we're going to look at another way to get rid of commits that you've already made.

If you have a commit earlier in your history, you can do an interactive rebase to drop it.

```
git log --oneline
git rebase -i HEAD~4
```

When you're rebasing, there's a few ways you can change your commit. We're going to drop right now but sometimes you'll want to edit or squash a commit to get a cleaner history.

## Anne Boleyn's execution, set up --force-with-lease
Henry's marriage to Anne was short-lived. After a few years, he accused her of adultery and treason and she was beheaded.

Remove `---A1[Anne Boleyn]`

```
git commit -m 'Executed Anne Boleyn'
git push
```

## Jane Seymour + 2nd Act of Succession, --force-with-lease

Eleven days later, Henry married Jane Seymour

Add `---J[Jane Seymour]`

```
git commit -m 'Married Jane Seymour'
```

Henry passed another act of succession that made Elizabeth illegitimate

We're going to interactive rebase again but we've already pushed this change up to our remote. So we're going to see how to overwrite this history as safely as possible.

```
git log --oneline
git rebase -i HEAD~3
  # drop 'Elizabeth was born'
git log --oneline
git push --force-with-lease
```

After a rebase, `git push` won't succeed. But `git push --force` is dangerous.

`--force-with-lease` is a safer alternative. It won't succeed if someone pushed commits that you don't have locally. It protects you from blowing away a coworker's changes and creating a ton of conflict headaches.

## Edward, git commit --amend
Later that year, Jane gave birth to Henry's first legitimate son, Edward. 

Add `E2[Edward]`

```
git commit -m 'Edward was born'
```


Unfortunately, she died due to complications of childbirth.

Remove `---J[Jane Seymour]`

```
git commit --amend
```

Jane Seymour was the love of Henry's life and he was eventually buried with her.

## Anne of Cleves, git checkout -p
Henry remained unmarried for a few years until political pressure made him take another bride. This time he married Anne of Cleves from Germany.

Add `---A2[Anne of Cleves]`

However even before the marriage, Henry was trying to get out of it. He never treated it like a real marriage and had it annulled 6 months later.

Since Anne didn't fight the annullment like Catherine had, she received the title of "The King's Sister" and the two remained friends for the rest of his life

Just like add, stash, and reset, we can use the patch flag with checkout as well

```
git checkout -p
```

## Catherine Howard, git reset HEAD~#

Henry was enamored with 17-yr-old Catherine Howard who had been a lady-in-waiting for Anne of Cleves. 

They were married shortly after that marriage dissovled

Add `---C2[Catherine Howard]`

```
git commit -m 'Married Catherine Howard'
```


But as with everything in Henry's world, that didn't last long.

She went the way of Anne Boleyn: was accused of adultery and beheaded.

If you haven't pushed a commit (or multiple commits) up to the remote, you can reset it.

This removes it from history and brings it back into the staging area.

```
git log --oneline
git reset HEAD~1
```

## Catherine Parr, git add -p

Henry's final wife was the total opposite of young Catherine Howard.

Catherine Parr was a widow two times over, extremely well educated, and a closet Protestant.

Add `---C3[Catherine Parr]`

At her urging, Henry passed his 3rd Act of Succession restoring Mary and Elizabeth to the line of succession

Add `M1[Mary]`

Add `E1[Elizabeth]`

```
git add -p
git commit -m 'Married Catherine Parr'
git add -p
git commit -m 'Restored Mary & Elizabeth to the line of succession'
```

## Full House of Tudor, git checkout branch from another file

All 3 of his children went on to serve as reigning monarch.

If you want to see what Henry's family tree looks like without all of his constant rewriting of history. I have a file in another branch with the full House of Tudor. 

Sometimes you don't want to cherry pick a whole commit but just one file from another branch.

```
git checkout full-tree full_tree.md
```