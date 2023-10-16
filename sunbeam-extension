#!/bin/sh

set -eu

# check if gh is installed
if ! command -v gh >/dev/null 2>&1; then
    echo "gh is not installed"
    exit 1
fi

if [ $# -eq 0 ]; then
sunbeam query -n '{
    title: "GitHub",
    commands: [
        {name: "search-repos", mode: "view", title: "Search Repositories"},
        {name: "list-prs", mode: "view", title: "List Pull Requests", params: [{name: "repo", type: "string", required: true}]}
    ]
}'
exit 0
fi

if [ "$1" = "search-repos" ]; then
    # shellcheck disable=SC2016
    QUERY=$(sunbeam query '.query')
    if [ "$QUERY" = "null" ]; then
        gh api "/user/repos?sort=updated" | sunbeam query '{
            type: "list",
            dynamic: true,
            items: map({
                title: .full_name,
                subtitle: (.description // ""),
                actions: [
                    { title: "Open in Browser", onAction: { type: "open", target: .html_url, exit: true }},
                    { title: "Copy URL", key: "o", onAction: {type: "copy",  text: .html_url, exit: true} },
                    { title: "List Pull Requests", key: "p", onAction: { type: "run", command: "list-prs", params: { repo: .full_name }}}
                ]
            })
        }'
        exit 0
    fi
    gh api "search/repositories?q=$QUERY" | sunbeam query '.items | {
        type: "list",
        dynamic : true,
        items: map({
                title: .full_name,
                subtitle: (.description // ""),
                actions: [
                    { title: "Open in Browser", onAction: { type: "open", target: .html_url, exit: true }},
                    { title: "Copy URL", key: "o", onAction: {type: "copy",  text: .html_url, exit: true} },
                    { title: "List Pull Requests", key: "p", onAction: { type: "run", command: "list-prs", params: { repo: .full_name }}}
                ]
            })
        }'
elif [ "$1" = "list-prs" ]; then
    REPOSITORY=$(sunbeam query -r '.params.repo')
    gh pr list --repo "$REPOSITORY" --json author,title,url,number | sunbeam query 'map({
        title: .title,
        subtitle: .author.login,
        accessories: [
            "#\(.number)"
        ],
        actions: [
            {title: "Open in Browser", onAction: { type: "open", target: .url, exit: true}},
            {title: "Copy URL", key: "c", onAction: { type: "copy", text: .url, exit: true}}
        ]
    }) | {type: "list", items: .}'
fi
