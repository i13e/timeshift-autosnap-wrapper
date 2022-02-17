#!/usr/bin/bash
# authors: foundobjects & i13e 
#
# this script prevents timeshift-autosnap from taking multiple snapshots 
# during the same update while using a supported AUR helper

# Automatically set which AUR helper to use
if [ -n "$AUR_HELPER" ] ; then
    echo "AUR_HELPER has been manually set to $AUR_HELPER by the user" # allow manually setting the AUR helper to be used
elif [ -x /usr/bin/paru ] ; then
    export AUR_HELPER='/usr/bin/paru' # if both are installed preference goes to paru
elif [ -x /usr/bin/yay ] ; then
    export AUR_HELPER='/usr/bin/yay' 
else
    echo "warning: please install a supported aur helper (paru or yay) to use this script." >&2
fi

# shellcheck disable=SC2155
export __AUTOSNAP_LOCK="$(mktemp -ut "aur-helper-autosnap.lock-XXXXXXX")"

# shellcheck disable=SC2064
trap "[[ -f $__AUTOSNAP_LOCK ]] && rm -f -- $__AUTOSNAP_LOCK" EXIT

# see hook /etc/pacman.d/hooks/00-timeshift-autosnap.hook for reference
exec "$(type -P $AUR_HELPER)" --sudoflags="--preserve-env=__AUTOSNAP_LOCK" "${@:--Syu}"
