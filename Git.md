git + :
commit 
branch name_of_new_branch point_to_place(HEAD by default)
checkout (-b : create a new branch and switch to that one)
switch
merge 
rebase
log
checkout(with the HEAD, not only branches) checkout hash_of_the_commit
^ (part of a relative reference to parent of the current HEAD value, but u can put a number after to choose the special parent, if there are a lot of them) ex: git checkout main^
~num_of_steps (works as ^)
^ and ~ can be mixed and repited in the "oneline" 
ex of forcing branch 3 commits back: git branch -f main HEAD~3
reset point_of_state_reset_to (for local repos, "time travelling")
revert point_of_reverting (for remote repos, new commit ahead with CHANGES, what we should do to get back in time) 
cherry-pick hash_of_commit1 hash_of_commit2 and so on
rebase -i point_of_reversing (manual reordering and working with commits, creates a new branch as fork to point of reversing)
commit --amend (replace current comment with new_configured one ) 
tag name_of_version point_of_tagging
describe reference_name(HEAD if nothing) (anything that can be assign to commit) (give you info about nearest ancestor tag to u curr pos) 

clone
fetch 
pull (alias for fetch+merge)
push (without arguments uses value of local config for push.default, which value is depending on the version of git and project setting so check it out//val=upstream was for lesson)
pull --rebase (short hand for fetch+rebase instead of fetch+merge) 
checkout -b totallyNotMain o/main (creates a new branch named totallyNotMain and sets it to track o/main)
git branch -u  o/main foo (force foo to track o/main) (without foo if already chechout foo) 
push origin main (remote source)
push origin source:destination (if u want to push not to the same specified branch on remote)
fetch origin foo
push origin source:destination (but here source - if from remote storage and destination is for local branch you want to fetch on)
push origin :foo (source is empty, so that command will delete foo branch on the remote storage)
fetch origin :foo (creates new branch foo on your local machine)
