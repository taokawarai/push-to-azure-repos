# push-to-azure-repos(ptar)

ローカルリポジトリから別のGitリモートリポジトリにPushをする際、`git remote add {REMOTE_NAME} {REPOS_URL}`でリモートリポジトリを登録して実行できる。
しかし、Azure Reposの場合は認証も絡んでくるのでPATの作成やコマンド実行時に認証情報を付与する必要がある。

特に、Windowsの資格情報マネージャでキャッシュが保存されている場合は`git config`で認証情報をあらかじめ用意しても参照されていなさそうな挙動をしたので、てこずった。

<img src="https://github.com/taokawarai/push-to-azure-repos/assets/35896206/7f50921f-bab9-4d48-b5b2-67d65b260db6" width=50%>

# 手作業で行う場合
1. リモートリポジトリのURLを確認
```
git remote -v
```
2. Azure Repos用のPATを作成：https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows#create-a-pat
3. PATをBase64でエンコードし、ヘッダー情報に付与してPush：https://learn.microsoft.com/ja-jp/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows#use-a-pat

ベースとなるコマンドは以下
```
git push {REMOTE_REPOSITORY_URL} {LOCAL_BRANCH}:{REMOTE_BRANCH}
```
上記コマンドにPATのヘッダー情報を付与して実行する。
```powershell
$MyPat = '{YOUR_PAT}'
$B64Pat = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("`:$MyPat"))
git -c http.extraHeader="Authorization: Basic $B64Pat" push {REMOTE_REPOSITORY_URL} {LOCAL_BRANCH}:{REMOTE_BRANCH}
```

# コマンドで行う
- https://learn.microsoft.com/ja-jp/rest/api/azure/devops/tokens/pats?view=azure-devops-rest-7.1
- ゴール：指定したAzure Reposリポジトリに対してPushを行う。
  - 準備：Push先のAzure ReposリポジトリのURLを取得
  - PATを作成
  - Pushを実行
  - PATを削除

---
- PATを作成：https://learn.microsoft.com/ja-jp/rest/api/azure/devops/tokens/pats/create?view=azure-devops-rest-7.1&tabs=HTTP#create-a-new-personal-access-token
```

PowerShellでのコマンドレットの作成とその配布は以下を参照
- https://github.com/Unitrends/unitrends-pstoolkit
$uri = 'https://functionapp20180430103816.azurewebsites.net/api/PostJsonFunction1'
$body = [System.Text.Encoding]::UTF8.GetBytes('{ "name":"値" }')

Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -Body $body -ContentType 'application/json'
```
