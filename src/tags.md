
<h1>How To Checkout Git Tags</h1>

History of commits.

```bash
git log --oneline --graph

```

Last 10 tags.
```bash
git describe --tags `git rev-list --tags --max-count=10`
```

Checkout
```bash
tag="v2.0"
git checkout $tag
```

<h1>Tag vs Branch</h1>
Both branches and tags refer to a commit. Their behavior, on the other hand, is different. The tag is associated with a single commit, whereas the branch changes from one commit to the next as new code is added.

The most significant distinction is in how they are used. When working on a feature, you use branches to organize code over time. Tagging, in general, refers to metadata linked with a build or deployment that is automatically or manually updated.
