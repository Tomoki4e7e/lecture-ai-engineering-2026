# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
  - 「3レベル」について、エージェントは宿題が「基礎・実装・発展」の
    3段階で構成され、どれか1つに取り組めばよいと回答した。
  - 回答の最後に「第1回」と関連する回を示した。
  - エージェントは回答前に用語集を検索するため、講座固有の内容を確認してから回答できる。
- `--trace` で見えたツール呼び出し（「LoRAとQLoRAの違いは？」）：
  - `search_glossary({"query": "LoRA"})`
  - `search_glossary({"query": "QLoRA"})`
  - それぞれの検索結果がモデルに返り、LoRA と QLoRA の違いを回答した。
- 1つの質問での LLM 呼び出し回数（`requests`）：
  - `2` 回（`tool_calls=2`）
- 追加したツールと、docstring に書いた説明：
  - `make_quiz(lecture: int) -> str` を [`app/tools.py`](../../app/tools.py)
    に実装した。
  - 指定回の用語を集めて1つを選び、「「用語」の意味は？」という問題と
    `definition` を JSON で返す。
  - 指定回の用語がない場合は、分かりやすいメッセージを返す。
  - [`app/agent.py`](../../app/agent.py) に `agent.tool_plain(tools.make_quiz)`
    を登録し、「確認問題を求められたら make_quiz を使う」という指示も追加した。

### 手順5の確認結果

- `uv run python -m app.agent --list-tools`
  - `search_glossary`、`get_lecture_summary`、`make_quiz` の3個が表示された。
- `uv run python -m app.agent --trace "第2回の内容から確認問題を1問出して"`
  - `make_quiz({"lecture": 2})` が呼び出された。
  - `make_quiz` の JSON 結果がモデルに返り、第2回の確認問題が出力された。
  - `requests=2`、`tool_calls=1` だった。
- `uv run pytest -q`
  - **11 passed**
- 追加の直接確認：
  - `make_quiz(2)` は `question` と `answer` を含む JSON を返した。
  - `make_quiz(0)` は「第0回の用語が見つかりません。」を返した。
- 構文チェック：
  - [`app/tools.py`](../../app/tools.py) に構文エラーなし。
- 作業ブランチ：
  - `lec03` を確認した。既存の作業ツリーを保護するため、`git pull` と
    ブランチ作成コマンドは再実行していない。

## 宿題

- 取り組んだレベル：基礎
- やったこと：
  - [`app/agent.py`](../../app/agent.py) の `INSTRUCTIONS` に、
    「比較を求められた回答は、『A』『B』『違い』の3項目の箇条書きで、
    簡潔にまとめる」という指示を追加した。
  - 同じ質問 `LoRAとQLoRAの違いは？` を、変更前後とも
    `uv run python -m app.agent --trace` で実行した。
- 結果：
  - 変更前は、LoRA と QLoRA の説明を段落形式で出力し、最後に
    「関係する回：第2回」と表示した。
  - 変更後は、`LoRA`、`QLoRA`、`違い` の3項目の箇条書きになり、
    指示どおり比較内容を簡潔に整理した。最後の関連回は「第2回」と表示された。
  - ツール呼び出しは変更前後とも
    `search_glossary({"query": "LoRA"})` と
    `search_glossary({"query": "QLoRA"})` の2回で変わらなかった。
  - LLM 呼び出し回数も変更前後とも `requests=2`、
    ツール呼び出し数も `tool_calls=2` だった。
  - 指示変更後もテストは **11 passed**、構文エラーなしだった。
- 気づき・疑問：
  - `INSTRUCTIONS` はツールの選択条件だけでなく、ツール結果を受け取った
    あとの回答形式にも影響する。
  - 回答形式を変えても、用語検索が必要という既存の指示は維持されたため、
    ツール名・引数・呼び出し回数は変わらなかった。
