---
layout: post
title: How to Remove GitHub Pages commits 

---

If you are using github pages to host your static blog, probably you don't need to keep commits to the blog repository. Usually, all commits are articles pushed to the repository or typo mistakes commits. I personally rather to remove them.

<!-- more -->

**To do that you need to use following git commands**

#### Check out to a temporary branch:
```git checkout --orphan TEMP_BRANCH```

#### Add all the files:

```git add -A```

#### Commit the changes:

```git commit -am "Initial commit"```

#### Delete the old branch:
```git branch -D master```

#### Rename the temporary branch to master:
```git branch -m master```

#### Finally, force update to our repository:
```git push -f origin master``` 