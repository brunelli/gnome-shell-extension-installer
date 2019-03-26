#!/bin/bash

   #################################################################
   #                                                               #
   #             GNOME Shell Extension Installer 1.6.2             #
   #                                                               #
   #       Copyright (C) 2017 Ian Brunelli <ian@brunelli.me>       #
   #       Licensed under the GNU General Public License 2.0       #
   #                                                               #
   #  https://github.com/brunelli/gnome-shell-extension-installer  #
   #                                                               #
   #################################################################

usage_guide() {
  cat << EOF
Usage: $SCRIPT_NAME EXTENSION_ID [EXTENSION_ID...] [GNOME_VERSION] [OPTIONS]

Options:
  -s or --search [STRING]	Interactive search.
  --yes 			Skip all prompts.
  --no-install 			Saves the extension(s) in the current directory.
  --update 			Check for new versions.
  --restart-shell 		Restart Shell after installing all extensions.
  -h or --help 			Print this message.

Usage examples:
  $SCRIPT_NAME 307			# Install "Dash to Dock"
  $SCRIPT_NAME 307 3.8		# Install for Shell 3.8
  $SCRIPT_NAME 53 --no-install	# Download "Pomodoro"
  $SCRIPT_NAME -s "User Themes"	# Search "User Themes"
EOF
}

enable_extension() {
  EXTL_OUTPUT=$( gnome-shell-extension-tool --enable-extension="$2" 2>&1 )
  sed -n "s/^'.*' is\( now\)\?\(.*\)\.$/[$1] Extension\2/p" <<< "$EXTL_OUTPUT"
}

extract_extension() {
  [[ -f "$EXTENSIONS_PATH/$EXTENSION_UUID/metadata.json" ]] && CODE=7 || CODE=0
  mkdir -p "$EXTENSIONS_PATH/$2"
  echo "[$1] Extracting extension"
  unzip -qq -o "$TMP_DIR/shell-$SELECTED_VERSION-extension-install.$3.zip" -d "$EXTENSIONS_PATH/$2"
  rm -f "$TMP_DIR/shell-$SELECTED_VERSION-extension-install.$3.zip"
  return $CODE
}

install_version() {
  [ "$INSTALL" == "download" ] && { move_file; return; }
  extract_extension "$EXTENSION_NAME" "$EXTENSION_UUID" "$EXTENSION_ID"
  case "$?" in
    0)
      enable_extension "$EXTENSION_NAME" "$EXTENSION_UUID" &&
      INSTALLED+=("$EXTENSION_ID")
      ;;
    7)
      UPDATED+=("$EXTENSION")
      ;;
  esac
}

move_file() {
  mv "$TMP_DIR/shell-$SELECTED_VERSION-extension-install.$EXTENSION_ID.zip" "$EXTENSION_UUID.shell-extension.zip" 2> /dev/null &&
  echo "[$EXTENSION_NAME] Extension saved as $EXTENSION_UUID.shell-extension.zip" ||
  echo -e "\e[0;31mERROR\e[0m: Failed to save file" >&2
  rm -f "$TMP_DIR/shell-$SELECTED_VERSION-extension-install.$EXTENSION_ID.zip"
}

download_version() {
  DOWNLOAD_TARGET=$( sed "s/^.*$2\": {\"pk[\": ]\+\([0-9]\+\),.*$/\1/" <<< "$1" )
  METADATA_FILE="$EXTENSIONS_PATH/$EXTENSION_UUID/metadata.json"
  [ -f "$METADATA_FILE" ] &&
  { VERSION_TARGET=$( sed "s/^.*$2\": {\"pk[\": ]\+[0-9]\+[\", ]\+version[\": ]\+\([0-9]\+\)}.*$/\1/" <<< "$1" )
    VERSION_INSTALLED=$( sed -n "s/^[[:space:]]*\"version[\": ]\+\([0-9]\+\).*$/\1/p" "$METADATA_FILE" )
    [[ $VERSION_INSTALLED ]] ||
    { echo -e "\e[0;31mERROR\e[0m: Use your package manager to update this extension" >&2
      return 1; }
    [[ $VERSION_TARGET -le $VERSION_INSTALLED ]] &&
    { if [ $UPDATE_MODE ]; then
        echo "[$EXTENSION_NAME] The extension is up-to-date"
        return 2
      fi; } ||
    echo "[$EXTENSION_NAME] New version available"; }
  echo "[$EXTENSION_NAME] Downloading extension"
  if curl $DOWNLOAD_PARAMETERS -o "$TMP_DIR/shell-$SELECTED_VERSION-extension-install.$EXTENSION_ID.zip" \
          "$EXTENSIONS_SITE/download-extension/$EXTENSION_UUID.shell-extension.zip?version_tag=$DOWNLOAD_TARGET"
  then
    return 0
  else
    echo -e "\e[0;31mERROR\e[0m: Failed to download extension" >&2
    return 1
  fi
}

check_version_availability() {
  unset SELECTED_VERSION
  for VERSION in ${EXTENSION_VERSIONS[@]}; do
    [ "$1" == "$VERSION" ] &&
    SELECTED_VERSION="$1"
  done
  [[ ! $SELECTED_VERSION ]] &&
  { TARGET=$( sed -n "s/^3\.\([0-9]\+\).*$/\1/p" <<< "$1" )
    LATEST=$( sed -n "s/^3\.\([0-9]\+\).*$/\1/p" <<< "${EXTENSION_VERSIONS[0]}" )
    [[ ${1%%\.*} == ${EXTENSION_VERSIONS[0]%%\.*} ]] &&
    [[ $TARGET -gt $LATEST ]] &&
    [[ $LATEST -gt 20 ]] &&
    SELECTED_VERSION="${EXTENSION_VERSIONS[0]}" &&
    [[ ! $UPDATE_MODE ]] &&
    echo -e "\e[0;33mWARNING\e[0m: Extension not available for GNOME Shell $1, using $SELECTED_VERSION instead" >&2; }
}

select_version() {
  if [[ "${EXTENSION_VERSIONS[0]}" ]]; then
    while [[ ! $SELECTED_VERSION ]]; do
      echo -n $1"Type a version to $INSTALL: "
      if [[ ! $SKIP_PROMPTS ]]; then
        read INPUT
        check_version_availability "$INPUT"
        [[ $INPUT =~ (q|quit|exit) ]] && return 1
        [[ ! $SELECTED_VERSION ]] &&
        echo "Available versions: ${EXTENSION_VERSIONS[@]}"
      else
        echo "${EXTENSION_VERSIONS[0]}"
        SELECTED_VERSION="${EXTENSION_VERSIONS[0]}"
      fi
    done
  else
    echo $1"No version available"
  fi
}

get_other_version() {
  [ "$EXTENSION_VERSIONS" ] &&
  { echo -e "\nThis extension is available for the following versions of GNOME Shell:"
    for VERSION in ${EXTENSION_VERSIONS[@]}; do
      echo "- $VERSION"
    done
    select_version; }
  echo
  [ "$SELECTED_VERSION" ] &&
  download_version "$EXTENSION_INFO" "$SELECTED_VERSION"
}

extract_info() {
  EXTENSION_NAME=$( sed 's/^.*\"name\"[: \"]*\([^\"]*\).*$/\1/' <<< "$1" )
  EXTENSION_DESCRIPTION=$( sed 's/^.*\"description\": \"//g' <<< "$1" |
                           sed 's/\", \"[a-z]\+\".*$//g' |
                           sed 's/\\\"/\"/g' )
  EXTENSION_CREATOR=$( sed 's/^.*\"creator\"[: \"]*\([^\"]*\).*$/\1/' <<< "$1" )
  EXTENSION_UUID=$( sed 's/^.*\"uuid\"[: \"]*\([^\"]*\).*$/\1/' <<< "$1" )
  EXTENSION_ID=$( sed 's/^.*\"pk\"[: \"]*\([^\"]*\),.*$/\1/' <<< "$1" )
  EXTENSION_LINK=$( sed 's/^.*\"link\"[: \"]*\([^\"]*\).*$/\1/' <<< "$1" )
  EXTENSION_URL=$( grep "download_url" <<< "$1" |
                   sed 's/^.*\"download_url\"[: \"]*\([^\"]*\).*$/\1/' )
  EXTENSION_VERSIONS=($( sed 's/[\"]*:[ ]*{[\"]*pk[\"]*:/\n/g' <<< "$1" |
                         sed '$ d' | sed 's/^.*\"//g' | sort -rV ))
}

download_info() {
  unset EXTENSION_INFO EXTENSIONS_QUERY EXTENSION_COMMENTS
  PAGES=-1
  TOTAL=-1
  EXTENSION_INFO=$( curl $DOWNLOAD_PARAMETERS "$EXTENSIONS_SITE${1// /%20}" )
  case "$?" in
    0)
      if [ "$( echo $EXTENSION_INFO | grep name )" ]; then
        return 0
      else
        echo -e "\e[0;31mERROR\e[0m: $2" >&2
        return 2
      fi
      ;;
    22)
      echo -e "\e[0;31mERROR\e[0m: $3 could not be found" >&2
      return 22
      ;;
    130)
      return 1
      ;;
    *)
      echo -e "\e[0;31mERROR\e[0m: $4 (curl error $?)" >&2
      return 1
      ;;
  esac
}

extract_comments() {
  [[ $# -ge 1 ]] && echo -e "[$1]\n\n"
  IFS=$'\n' read -d '' -r -a COMMENTS_QUERY <<< "$( echo $EXTENSION_COMMENTS | cut -f1 --complement -d"[" | sed 's/, {/\n/g' )"

  for COMMENT in "${COMMENTS_QUERY[@]}"; do
    LINE=$( perl -ne '
      foreach $a (qw/username is_extension_creator standard rating comment/) {
       /"($a)"\s*:\s*("((?:[^"\\]|\\.)*)"|[a-z0-9]+)/;
       if ($1 eq "username") {
         print "\e[1;34m$3\e[0m"
       } elsif ($1 eq "is_extension_creator") {
         if ($2 eq "true") { print " [A]" }
       } elsif ($1 eq "standard") {
         print " ($3):\\n"
       } elsif ($1 eq "rating") {
         print "Rating: $2\\n"
       } elsif ($1 eq "comment") {
         print $s = $3 =~ s/\\\"/\"/gr, "\\n\\n";
       }
      }' <<< "$COMMENT" )
    echo -e "$LINE"
  done
}

download_comments() {
  unset EXTENSION_COMMENTS
  EXTENSION_COMMENTS=$( curl $DOWNLOAD_PARAMETERS "$EXTENSIONS_SITE$1" )
  case "$?" in
    0)
      if [ "$( echo $EXTENSION_COMMENTS | grep comment )" ]; then
        return 0
      else
        echo -e "\n$2\n"
        return 2
      fi
      ;;
    22)
      echo -e "\e[0;31mERROR\e[0m: $3 could not be found" >&2
      return 22
      ;;
    130)
      return 1
      ;;
    *)
      echo -e "\e[0;31mERROR\e[0m: $4 (curl error $?)" >&2
      return 1
      ;;
  esac
}

search_help() {
  cat << EOF
<number(s)>	${INSTALL^} extension(s)
c<number>	Display comments
d<number(s)>	Get description(s)
l<number(s)>	Get link(s) on extensions.gnome.org
p<number>	Go to page
r		Print the search content again
sn		Sort by name
sr		Sort by recent
sd		Sort by downloads
sp		Sort by popularity (default)
/<string>	Perform another search
home		Load extensions.gnome.org homepage
h or help	Show this message
q or quit	Exit search shell
EOF
}

load_content() {
  unset SEARCH_CONTENT
  PAGES=$( sed 's/^.*numpages[\": ]*\([^\"]*\),.*$/\1/' <<< "$EXTENSION_INFO" )
  TOTAL=$( sed 's/^.*total[\": ]*\([^\"]*\),.*$/\1/' <<< "$EXTENSION_INFO" )
  SEARCH_CONTENT=$( echo "Displaying $TOTAL item(s). Page $PAGE_NUM of $PAGES.\n")
  IFS=$'\n' read -d '' -r -a EXTENSIONS_QUERY <<< "$( echo $EXTENSION_INFO | cut -f1 --complement -d"[" | sed 's/{[\"]*shell_version_map[\"]*:/\n/g' )"

  COUNTER=0
  for EXTENSION in "${EXTENSIONS_QUERY[@]}"; do
    extract_info "$EXTENSION"
    SEARCH_CONTENT+=$( echo "\n$COUNTER: $EXTENSION_NAME, by $EXTENSION_CREATOR" \
                       "\n   Versions: " )
    [[ "${EXTENSION_VERSIONS[@]}" ]] &&
    SEARCH_CONTENT+=$( echo "${EXTENSION_VERSIONS[@]}" ) ||
    SEARCH_CONTENT+=$( echo "(no version available)" )
    (( COUNTER++ ))
  done
  SEARCH_CONTENT+=$( echo "\n" )
  echo -e "$SEARCH_CONTENT"
}

restart_shell() {
  [[ $( pgrep gnome-shell ) ]] || return
  echo "Restarting GNOME Shell..."
  dbus-send --session --type=method_call \
            --dest=org.gnome.Shell /org/gnome/Shell \
            org.gnome.Shell.Eval string:"global.reexec_self();"
}

interactive_search() {
  echo -e "Type \"help\" to get information on how to use the search."
  while IFS='' read -r -e -d $'\n' -p "Enter a command: " COMMAND; do
    trap echo SIGINT
    if [[ $COMMAND =~ ^[0-9]+( |[0-9])*$ ]]; then
      for n in $COMMAND; do
        if extract_info "${EXTENSIONS_QUERY[$n]}" && [ $n -lt $TOTAL ]; then
          [[ ${#EXTENSION_VERSIONS[@]} == 1 ]] &&
          SELECTED_VERSION="${EXTENSION_VERSIONS[@]}"
          check_version_availability "$GNOME_VERSION"
          [ "$SELECTED_VERSION" ] ||
          select_version "[$EXTENSION_NAME] "
          [ "$SELECTED_VERSION" ] &&
          download_version "${EXTENSIONS_QUERY[$n]}" "$SELECTED_VERSION" &&
          install_version
        else
          echo "[$n] Invalid extension number" >&2
        fi
      done
    elif [[ $COMMAND =~ ^c( )*[0-9]+$ ]]; then
      if [[ ${COMMAND:1} -lt $TOTAL ]]; then
        command -v gnome-shell > /dev/null ||
        { echo -e "\e[0;31mERROR\e[0m: Comments can't be parsed, perl is not installed." >&2
          continue; }
        extract_info "${EXTENSIONS_QUERY[${COMMAND:1}]}"
        echo "[$EXTENSION_NAME] Getting comments"
        download_comments "/comments/all/?pk=$EXTENSION_ID&all=true" \
                          "No comment to display" \
                          "The page" \
                          "Failed to get comments" &&
        { if [[ $( wc -c <<< "$EXTENSION_COMMENTS" ) -gt 2000 ]] &&
             [[ $( type less 2> "/dev/null" ) ]]; then
            extract_comments "$EXTENSION_NAME" | less -R
          else
            echo -e "\n"; extract_comments; echo -e "\n"
          fi; }
      else
        echo "[${COMMAND:1}] Invalid extension number" >&2
      fi
    elif [[ $COMMAND =~ ^d([0-9 ])+$ ]]; then
      for n in ${COMMAND:1}; do
        if [[ $n -lt $TOTAL ]]; then
          extract_info "${EXTENSIONS_QUERY[$n]}"
          if [[ $( echo -e "$EXTENSION_DESCRIPTION" | wc -l ) -gt 10 ]] &&
             [[ $( type less 2> "/dev/null" ) ]]; then
            less <<< "$( echo -e "[$EXTENSION_NAME]\n$EXTENSION_DESCRIPTION" )"
          else
            echo -e "[$EXTENSION_NAME]\n$EXTENSION_DESCRIPTION"
          fi
        else
          echo "[$n] Invalid extension number" >&2
        fi
      done
    elif [[ $COMMAND =~ ^l([0-9 ])+$ ]]; then
      for n in ${COMMAND:1}; do
        if [[ $n -lt $TOTAL ]]; then
          extract_info "${EXTENSIONS_QUERY[$n]}" &&
          echo "[$EXTENSION_NAME] $EXTENSIONS_SITE$EXTENSION_LINK" ||
          echo "[$n] Invalid extension number" >&2
        else
          echo "[$n] Invalid extension number" >&2
        fi
      done
    elif [[ $COMMAND =~ ^p( )*[0-9]+$ ]]; then
      PAGE="$( echo ${COMMAND:1} | sed 's/^0*//' )"
      if [[ $PAGE -le $PAGES ]]; then
        PAGE_NUM="${COMMAND:1}"
        [[ $SEARCH_STRING ]] &&
        echo "[$SEARCH_STRING] Obtaining page $PAGE_NUM" ||
        echo "Obtaining page $PAGE_NUM"
        download_info "/extension-query/?sort=$SORT&search=$SEARCH_STRING&page=$PAGE_NUM" \
                      "No items to display" \
                      "The page $PAGE_NUM" \
                      "Failed to obtain page info" &&
        load_content
      else
        echo "[$PAGE] Invalid page number" >&2
      fi
    elif [[ $COMMAND =~ ^s(n|r|d|p)$ ]]; then
      case ${COMMAND:1:1} in
        n) SORT="name" ;;
        r) SORT="recent" ;;
        d) SORT="downloads" ;;
        p) SORT="popularity" ;;
      esac
      echo "Sorting by $SORT"
      [ $TOTAL -gt 0 ] &&
      { [[ $SEARCH_STRING ]] &&
        echo "[$SEARCH_STRING] Obtaining page 1" ||
        echo "Obtaining page 1"
        download_info "/extension-query/?sort=$SORT&search=$SEARCH_STRING&page=1" \
                     "No items to display" \
                     "The page $PAGE_NUM" \
                     "Failed to obtain page info" &&
        load_content; }
    elif [[ $COMMAND =~ ^(h|help)$ ]]; then
      search_help
    elif [[ $COMMAND =~ ^home$ ]]; then
      unset SEARCH_STRING
      echo "Loading homepage"
      download_info "/extension-query/?sort=$SORT&page=1" \
                    "No items to display" \
                    "The page" \
                    "Failed to obtain page info" &&
      load_content
    elif [[ $COMMAND =~ ^r$ ]]; then
      [ $TOTAL -gt 0 ] &&
      echo -e "$SEARCH_CONTENT"
    elif [[ $COMMAND =~ ^(exit|quit|q)$ ]]; then
      break
    elif [[ $COMMAND =~ ^/ ]]; then
      if [ "${COMMAND:1}" ]; then
        SEARCH_STRING="${COMMAND:1}"
        echo "[$SEARCH_STRING] Performing search"
        download_info "/extension-query/?sort=$SORT&search=$SEARCH_STRING&page=1" \
                      "No items to display" \
                      "The search page for \"$SEARCH_STRING\"" \
                      "Failed to obtain page info" &&
        load_content
      else
        echo "No search string specified" >&2
      fi
    elif [[ ! $COMMAND ]]; then
      continue
    else
      echo "Unknown command. Type h for help." >&2
    fi
    trap - SIGINT
    history -s "$COMMAND"
  done
}

get_info_from_id() {
  echo "[$1] Obtaining extension info"
  download_info "/extension-info/?pk=$1" \
                "Blank file" \
                "The extension $1" \
                "Failed to obtain extension info" &&
  extract_info "$EXTENSION_INFO"
}

get_extension() {
    check_version_availability "$1"
    { if [ "$SELECTED_VERSION" ]; then
        download_version "$EXTENSION_INFO" "$SELECTED_VERSION"
      else
        echo "[$EXTENSION_NAME] Extension not available for GNOME Shell $1"
        get_other_version "$1"
      fi; } &&
    install_version
}

for CMD in curl dbus-send
do
    command -v "$CMD" > /dev/null ||
    { echo "Missing required command: ${CMD}" >&2
      exit 1; }
done

GNOME_VERSION=$( gnome-shell --version 2> /dev/null |
                 sed -n "s/^.* \([0-9]\+\.[0-9]\+\).*$/\1/p" )
DOWNLOAD_PARAMETERS="-Lfs"
EXTENSIONS_SITE="https://extensions.gnome.org"
SORT="popularity"

[[ $EUID -eq 0 ]] &&
EXTENSIONS_PATH="/usr/share/gnome-shell/extensions" ||
{ EXTENSIONS_PATH="$HOME/.local/share/gnome-shell/extensions"
  mkdir -p "$EXTENSIONS_PATH"; }
command -v gnome-shell > /dev/null &&
INSTALL="install" ||
{ INSTALL="download"
  echo -e "\e[0;33mWARNING\e[0m: Install is not available" >&2; }
[[ -w "/tmp" ]] && TMP_DIR="/tmp" || TMP_DIR="."
SCRIPT_NAME=$( basename "$0" )
RESTART=0
PAGES=-1
TOTAL=-1
unset INSTALL_ARRAY INSTALLED UPDATED SEARCH_STRING SKIP_PROMPTS UPDATE_MODE

while [ $# -gt 0 ]; do
  if [[ $1 =~ ^-(h|-help)$ ]]; then
    usage_guide
    exit 0
  elif [[ $1 =~ ^-(s|-search)$ ]]; then
    [[ -n ${SEARCH_STRING+ } ]] &&
    { echo -e "$SCRIPT_NAME: excessive arguments" >&2
      exit 2; }
    if [[ ! $2 =~ ^- ]] && [[ ! $2 =~ ^[0-9\.]+$ ]]; then
      SEARCH_STRING="$2"
      shift 1
    else
      SEARCH_STRING=""
    fi
    shift 1
  elif [ "$1" == "--yes" ]; then
    SKIP_PROMPTS=1
    shift
  elif [ "$1" == "--no-install" ]; then
    INSTALL="download"
    shift
  elif [ "$1" == "--update" ]; then
    UPDATE_MODE=1
    shift
  elif [ "$1" == "--restart-shell" ]; then
    if [[ "$XDG_SESSION_TYPE" == "wayland" ]]; then
      echo -e "\e[0;33mWARNING\e[0m: Restart is not available on Wayland" >&2
      RESTART=-1
    else
      RESTART=1
    fi
    shift
  elif [[ $1 =~ ^3\.[0-9\.]+$ ]]; then
    GNOME_VERSION="$1"
    shift
  elif [[ $1 =~ ^[0-9]+$ ]]; then
    INSTALL_ARRAY+=( $1 )
    shift
  elif [[ $1 =~ ^- ]]; then
    echo -e "$SCRIPT_NAME: unrecognized option '$1'\n" >&2
    usage_guide >&2
    exit 3
  else
    echo "$SCRIPT_NAME: '$1' is not a valid Extension ID" >&2
    exit 4
  fi
done

[[ $GNOME_VERSION ]] ||
{ echo -e "\e[0;31mERROR\e[0m: You need to pass a GNOME version as argument" >&2
  exit 5; }

if [[ -n ${SEARCH_STRING+ } ]]; then
  [[ ${#INSTALL_ARRAY[@]} -gt 0 ]] &&
  { echo -e "$SCRIPT_NAME: excessive arguments" >&2
    exit 2; }
  unset SKIP_PROMPTS
  PAGE_NUM="1"
  [[ $SEARCH_STRING ]] &&
  { echo "[$SEARCH_STRING] Performing search"
    download_info "/extension-query/?sort=$SORT&search=${SEARCH_STRING// /%20}&page=1" \
                  "No items to display" \
                  "The search page for \"$SEARCH_STRING\"" \
                  "Failed to obtain page info" &&
    load_content; }
  interactive_search
elif [ ${#INSTALL_ARRAY[@]} -gt 0 ]; then
  for EXTENSION in ${INSTALL_ARRAY[@]}; do
    get_info_from_id "$EXTENSION" &&
    get_extension "$GNOME_VERSION"
  done
elif [[ $UPDATE_MODE ]]; then
  UPDATE_LIST=($( find -L "$EXTENSIONS_PATH" -maxdepth 2 -name "metadata.json" \
                       -exec sed -n "s/^[[:space:]]*\"uuid[\": ]\+\(.*\)\".*$/\1/p" {} \; ))
  for EXTENSION in ${UPDATE_LIST[@]}; do
    IS_GIT_DIR=$( type git > /dev/null 2> /dev/null &&
                  git -C "$EXTENSIONS_PATH/$EXTENSION/" rev-parse --is-inside-work-tree 2> /dev/null )
    if [[ $IS_GIT_DIR ]]; then
      echo "[$EXTENSION] Updating from git"
      GIT_UPDATE=$( LANG=en_US git -C "$EXTENSIONS_PATH/$EXTENSION/" pull --rebase=false 2>&1 )
      case "$GIT_UPDATE" in
        "Already up"*)
          echo "[$EXTENSION] The extension is up-to-date"
          ;;
        "")
          echo -e "\e[0;31mERROR\e[0m: Failed to update extension (git exit code $?)" >&2
          ;;
        *)
          echo "[$EXTENSION] Extension updated"
          cat <<< "$GIT_UPDATE"
          UPDATED+=("$EXTENSION")
          ;;
      esac
    else
      echo "[$EXTENSION] Searching extensions.gnome.org"
      download_info "/extension-query/?search=${EXTENSION}" \
                    "Extension not found" \
                    "The search page for \"$SEARCH_STRING\"" \
                    "Failed to obtain extension info" &&
      { IFS=$'\n' read -d '' -r -a EXTENSIONS_QUERY <<< "$( echo $EXTENSION_INFO | cut -f1 --complement -d"[" | sed 's/{[\"]*shell_version_map[\"]*:/\n/g' )"
        unset EXTENSION_INFO
        for SEARCH_RESULT in "${EXTENSIONS_QUERY[@]}"; do
          extract_info "$SEARCH_RESULT"
          [[ "$EXTENSION" == "$EXTENSION_UUID" ]] &&
          { EXTENSION_INFO="$SEARCH_RESULT"; break; }
        done
        [[ $EXTENSION_INFO ]] && get_extension "$GNOME_VERSION"; }
    fi
  done
else
  usage_guide
fi

[[ ${#INSTALLED[@]} -gt 0 ]] && echo -n "${#INSTALLED[@]} new extension(s) installed. "
[[ ${#UPDATED[@]} -gt 0 ]] && echo -n "${#UPDATED[@]} extension(s) updated. "
if [[ ${#INSTALLED[@]} -gt 0 ]] || [[ ${#UPDATED[@]} -gt 0 ]]; then
  if [[ $RESTART == 1 ]]; then
    restart_shell
  elif [[ -n ${SEARCH_STRING+ } ]] && [[ "$XDG_SESSION_TYPE" != "wayland" ]]; then
    echo -n "Restart GNOME Shell? (y/N) "
    read RESTART
    [[ $RESTART =~ ^(y|Y) ]] && restart_shell
  else
    echo "You may have to restart GNOME Shell."
  fi
fi
