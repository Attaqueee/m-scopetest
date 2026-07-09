pr from fork


BASE="QueComo/m-scopetest"
FORK="$GITHUB_REPOSITORY"
AUTH="Authorization: Bearer $GITHUB_TOKEN"
ACC="Accept: application/vnd.github+json"

echo "================ BLOCK 1: identity and context ================"
echo "GITHUB_REPOSITORY = $FORK"
echo "GITHUB_USER       = $GITHUB_USER"
echo -n "token /user login = "; curl -s -H "$AUTH" https://api.github.com/user | jq -r '.login'

echo; echo "================ BLOCK 2: the misleading role echo ================"
echo "GET /repos/$BASE  ->  .permissions:"
curl -s -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$BASE" | jq '.permissions'

echo; echo "================ BLOCK 3: BASE write/admin probes (the answer) ================"
curl -s -o /dev/null -w "  [A] BASE list secrets (needs ADMIN): HTTP %{http_code}\n" \
  -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$BASE/actions/secrets"
BASE_DEF=$(curl -s -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$BASE" | jq -r '.default_branch')
BASE_SHA=$(curl -s -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$BASE/git/refs/heads/$BASE_DEF" | jq -r '.object.sha')
curl -s -o /dev/null -w "  [B] BASE create-ref (needs WRITE): HTTP %{http_code}\n" \
  -X POST -H "$AUTH" -H "$ACC" \
  -d "{\"ref\":\"refs/heads/scopetest-probe\",\"sha\":\"$BASE_SHA\"}" \
  "https://api.github.com/repos/$BASE/git/refs"

echo; echo "================ BLOCK 4: CONTROL - same token on the FORK ================"
FORK_DEF=$(curl -s -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$FORK" | jq -r '.default_branch')
FORK_SHA=$(curl -s -H "$AUTH" -H "$ACC" "https://api.github.com/repos/$FORK/git/refs/heads/$FORK_DEF" | jq -r '.object.sha')
curl -s -o /dev/null -w "  [C] FORK create-ref (needs WRITE): HTTP %{http_code}\n" \
  -X POST -H "$AUTH" -H "$ACC" \
  -d "{\"ref\":\"refs/heads/scopetest-probe\",\"sha\":\"$FORK_SHA\"}" \
  "https://api.github.com/repos/$FORK/git/refs"
echo "==========================================================="
