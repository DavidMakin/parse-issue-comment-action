## parse-issue-comment-action

This is a Github action, highly infuenced from https://github.com/peter-murray/issue-body-parser-action. The main use case 
is for this action to be triggered by a Github issue comment in order to parse a JSON or YAML payload from it. 

## Inputs

## `github_token`

**Required** The Github token to use for getting the issue comment via the API.

## `comment_id`

**Required** The id of the issue comment that will be parsed.

## `format`

**Required** The format of the payload we expect to be parsing from the comment. Defaults to `json`

## `marker`

**Required** The optional marker for the YAML or JSON payload to parse, in case the comment contains multiple. Defaults to "".

## `fail_on_missing`

If true, the action will fail if the payload is not found in the Comment body

## Outputs

## `content`

The data that was parsed from the JSON or YAML payload as a string. It can be easily converted to JSON using the [fromJSON expression](https://docs.github.com/en/enterprise-cloud@latest/actions/learn-github-actions/expressions#fromjson) 

## Example usage
``` yaml
on:
  issue_comment:
    types: [created]
    
jobs:
  do_something:
    name: A job that does something
    runs-on: ubuntu-latest
    steps:
      - name: Parse comment 
        id: parse
        uses: actions/parse-issue-comment-action@v0.1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          comment_id: ${{ github.event.comment.id }}
          format: yaml
      - name: Echo the output
        run: echo ${{ steps.parse.outputs.content }} 
      - name: Use the output as JSON
        run: echo ${{ fromJson(steps.parse.outputs.content).some_key }}
```

### Fail on missing payload
By default the action will fail if the payload is not found in the issue. This behavior can be changed by setting the `fail_on_missing` option to `false`.

```yaml
name: Parse comment
uses: jkakavas/parse-issue-comment-action@v0.1
id: parse
with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    comment_id: ${{ github.event.comment.id }}
    fail_on_missing: false
```

In this case, when the payload is not found in the issue, the payload value will be set to `NOT_FOUND`. You may want to skip other actions in your workflow if this is the case by using an `if` conditional as follows:
```yaml
name: Run something that uses the payload
    if : steps.parse.outputs.payload != 'NOT_FOUND'
    id: run_something
    env:
        SOME_KEY: ${{ fromJson(steps.parse.outputs.payload).some_key }}
    run: |
    echo $SOME_KEY
