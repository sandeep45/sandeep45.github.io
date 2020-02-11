install iterm2_shell_integration from iterm menu
ensure its enabled from your `.zshrc`
do `export ITERM2_SQUELCH_MARK=1` to disable its markers
add a script which does the following
```
function iterm2_print_user_vars() {
  iterm2_set_user_var badge $(basename "`pwd`")
}
then set badge to `\(user.badge)`
```
then load this script  from your `.zshrc`

```
test -e "${HOME}/.dir_badges.sh" && source "${HOME}/.dir_badges.sh"
``` 