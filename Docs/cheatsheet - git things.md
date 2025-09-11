# clean up a failed release/tag
Go to GitHub → Releases → delete the release you just created.
•	Then delete the tag that was created:
```bash
# locally
git tag -d vX.Y.Z       # delete locally
git push origin :refs/tags/vX.Y.Z   # delete remotely
```
Now fix the problem that caused the prod build to fail

And Re-release
```bash
# Use the GitHub UI to create a release again

# Or just use git
git tag vX.Y.Z
git push origin vX.Y.Z
```

## updating packages
npm install glob@^8 --save

## Throw away work and rollback main

```bash
git checkout main
git reset --hard v1.2.3   # replace with your tag
git push origin main --force
	•	This rewrites main so it looks like the merges never happened.
	•	Clean slate for you to redo the changes properly.
```

## api calls
```bash
$ curl https//localhost:3001/v1/version
$ curl https://api.staging.trippl.ca/v1/version
$ curl https://api.dev.trippl.ca/v1/version
$ curl https://api.app.trippl.ca/v1/version
```
