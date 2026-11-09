# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：エージェントを使うことによって正しい知識を獲得し、正しい答えが返ってきた。

- `--trace` で見えたツール呼び出し（ツール名と引数）：
--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…

- 1つの質問での LLM 呼び出し回数（`requests`）：2

- 追加したツールと、docstring に書いた説明：
import random

def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つを選び、確認問題と答えを返す。

    「第2回の確認問題を出して」のように、
    復習用の問題を求められたときに使う。lecture は1始まりの整数。
    """
    # 1. load_glossary() から lecture が一致する用語を集める
    terms = [e for e in load_glossary() if e["lecture"] == lecture]
    if not terms:
        return f"第{lecture}回の用語は用語集にありません。"

    # 2. 1つ選ぶ
    entry = random.choice(terms)

    # 3. 問題と答えを文字列で返す
    return json.dumps(
        {
            "question": f"「{entry['term']}」とは何ですか?",
            "answer": entry["definition"],
        },
        ensure_ascii=False,
    )

## 宿題

- 取り組んだレベル：基礎 / 実装 

- やったこと：
app/agent.py の INSTRUCTIONS や、app/tools.py のツールの docstring を書き換えた
実装課題として、学習記録を learning_log/ に保存する save_learning_log を作った。
保存前に利用者の確認を挟むため、requires_approval=True で登録し、output_type=[str, DeferredToolRequests] を指定した。
承認を聞いて続きを実行する run_with_approval を作り、main() の run_sync の呼び出しを置き換えた。

- 結果：
最初は run_sync を置き換えていなかったため、承認画面が出ず、ファイルも保存されなかった。trace では save_learning_log の tool call だけがあり、tool return がなかった。
main() の呼び出しを run_with_approval(target, ...) に置き換えた後は、[承認が必要] と [y/N] が表示される想定。yなら保存、nなら拒否される。

- 気づき・疑問：
requires_approval=True のツールは、承認されるまで実行されない。関数を定義しただけでは動かず、呼び出す側で承認の処理を書く必要がある。
モデルは save_learning_log を呼ぼうとするだけで、実際に実行するかどうかはプログラム側が握っている。取り消しにくい操作の安全装置として分かりやすい。
docstring に「頼まれたときだけ使う」と書くと、モデルが勝手に保存しにくくなる。