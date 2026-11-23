# Emotion Word Survey (English)

A three-part web survey built around 13 emotion words and 13 photos of facial expressions. It is a single `index.html` file, and responses are downloaded as a JSONL file on submission. This is the English version of [emotion-survey](https://github.com/yerinNam/emotion-survey); the Korean emotion words are shown with the English labels listed below.

Live site: https://yerinnam.github.io/emotion-survey-en/

## Running

Open `index.html` in a recent browser (Chrome, Edge, Safari or Firefox). No server or installation is needed and it works offline; without internet only the web font falls back to a system font, and the similarity scores are not sent to the Google Sheet (they are still in the JSONL file).

## Flow

1. **Start**: enter a Participant ID.
2. **Part 1, emotion similarity rating**: rate all 78 pairs of emotion words, one pair at a time, on a 1–10 slider. The slider must be moved before going to the next pair, and "Previous" goes back to change an answer. After the last pair the participant moves on to Part 2 and cannot return; at that point the scores are also sent to the Google Sheet tab `응답 결과(외국인)`.
3. **Part 2, grouping emotion words**: pick a colour, then tap word cards to colour them, so that words with similar meanings share a colour. All 13 cards must be coloured to continue, and there is no way back.
4. **Part 3, matching faces and words**: for every face, pick a 1st- and 2nd-choice word and rate confidence in each from 1 to 5.
   - While choosing, the same word may be used on several faces.
   - To submit, the 13 words must each be the 1st choice for exactly one face.
5. **Submit**: `emotion_survey_en_<ParticipantID>_<date-time>.jsonl` is downloaded.

Pair order (and left/right position within a pair), card order, face order and word-bank order are randomised per participant.

## Emotion words

| No. | Korean (CSV) | English label shown |
|---|---|---|
| 01 | 열망 | Longing / yearning |
| 02 | 불쾌함 | Displeasure / unease |
| 03 | 탐욕 | Material Greed |
| 04 | 옹졸함 | Pettiness / small-mindedness |
| 05 | 씁쓸함 | Bitterness |
| 06 | 불만족 | Dissatisfaction |
| 07 | 못마땅함 | Disapproval / disgruntlement |
| 08 | 시기 | Jealousy (begrudging someone's good fortune) |
| 09 | 분함 | Resentment |
| 10 | 탐냄 | Coveting / Wanting what someone else has |
| 11 | 언짢음 | Feeling really irritated |
| 12 | 낙담 | Discouragement |
| 13 | 소유욕 | Possessiveness / desire to own |

The labels live in `WORD_EN` in `src/survey_template.html`. All words in the output (pair keys, groups, choices, `target_word`) use these English labels.

## Output format (JSONL, 15 lines per participant)

Every line has `participant_id`, `session_id` and `part`.

**Line 1: `"part": "similarity"`**

| Field | Contents |
|---|---|
| `scores` | Score per pair. Keys are the 78 `"Word A - Word B"` pairs in fixed CSV order (same as the Google Sheet columns) |
| `ratings` | Responses in presentation order. Each has `order`, `pair_key`, `word1`/`word2` (left/right on screen), `score`, `duration_ms` (accumulated if revisited) |
| `n_pairs`, `scale_min`, `scale_max` | Number of pairs and scale range |
| `sheet_post` | Google Sheet send status: `sent` (request sent), `failed` (network error), `disabled` (no URL) |
| `started_at`, `finished_at`, `duration_sec`, `total_rating_ms` | Start/end time, duration in seconds, sum of per-pair response times |

**Line 2: `"part": "word_clustering"`**

| Field | Contents |
|---|---|
| `groups` | List of groups, each with `group_no`, `name` (participant-given, empty if none), `size`, `words` |
| `assignment` | Group number per word, e.g. `{"Bitterness": 1, …}` |
| `n_groups` | Number of groups |
| `card_order` | Card order shown on screen |
| `history` | Colouring/clearing log (`t_ms`, `action`, `word`, `group_id` …) |
| `started_at`, `finished_at`, `duration_sec` | Start, end, duration in seconds |

**Lines 3–15: `"part": "face_matching"` (one line per face)**

| Field | Contents |
|---|---|
| `face_no` | Number shown on screen (presentation order) |
| `image_id`, `image_file` | Image ID and original file name |
| `target_word`, `target_word_ko` | Intended emotion (English label and Korean word from the CSV) |
| `rank1_word`, `rank1_confidence` | 1st choice and confidence (1–5) |
| `rank2_word`, `rank2_confidence` | 2nd choice and confidence (1–5) |
| `rank1_correct` | Whether the 1st choice is the intended emotion |
| `target_in_top2` | Whether the intended emotion is the 1st or 2nd choice |
| `n_word_changes`, `n_confidence_changes` | Number of word and confidence changes |
| `history` | Log of placing/removing/swapping words on that face |
| `word_bank_order` | Word-bank order shown on screen |
| `started_at`, `submitted_at`, `duration_sec` | Part 3 start, submission time, duration in seconds |

Participant files can simply be concatenated into one dataset, e.g. `cat emotion_survey_en_*.jsonl > all.jsonl`, then read in Python with `pandas.read_json('all.jsonl', lines=True)` and split by `part`.

## Settings

Edit `CONFIG` in the `<script>` near the bottom of `src/survey_template.html`, then rebuild (below).

| Option | Default | Description |
|---|---|---|
| `shufflePairs` | `true` | Randomise similarity pair order and left/right position |
| `simMin`, `simMax`, `simDefault` | `1`, `10`, `1` | Similarity scale range and slider start |
| `sheetUrl`, `sheetName` |  | Apps Script web-app URL and sheet tab for similarity scores. Set `sheetUrl` to `''` to disable |
| `shuffleCards`, `shuffleImages`, `shuffleWords` | `true` | Randomise card, face and word-bank order |
| `confMin`, `confMax` | `1`, `5` | Confidence scale range |
| `confQuestion`, `confLabels` |  | Confidence question and end labels |
| `uniqueFirst` | `true` | Each word must be the 1st choice for exactly one face |
| `uniqueSecond` | `false` | 2nd choices must not repeat either |

## Sending responses to Notion

On submit, the whole JSONL can be sent to Notion as one child page per participant under "인지와 정서 - 데이터 수집 (en)" (the Korean version uses the (ko) page). A Notion token cannot go in a public page, so a Google Apps Script web app sits in between.

1. Create a new project at script.google.com and paste `apps_script/notion_relay.gs`.
2. Project Settings → Script properties: add `NOTION_TOKEN` = the Notion integration secret. Never put the token in code or HTML.
3. In Notion, add the integration to both pages (⋯ → Connections).
4. Deploy → New deployment → Web app; Execute as: Me; Who has access: Anyone.
5. Put the `/exec` URL in `CONFIG.notionUrl` in `src/survey_template.html` and run `python build.py`.

Sending uses `no-cors`, so the page cannot confirm success; the file downloaded on the participant's computer is the backup.

## Changing images or words (rebuild)

1. Replace the `NN_word.png` files and `00_감정단어_이미지_일람표.csv` in `stimuli/`. The number and Korean word in each file name must match the CSV `번호` and `감정단어` columns, and every Korean word needs an entry in `WORD_EN`.
2. Run the following to regenerate `index.html`:

```bash
pip install pillow
python build.py
```

Images are embedded as lossless WebP and checked pixel-for-pixel against the originals; resolution and quality are unchanged.

## Files

```
index.html                 # survey file for participants (images embedded, ~25 MB)
build.py                   # builds index.html from stimuli/
src/survey_template.html   # survey UI and logic (without images)
stimuli/                   # 13 original face images and the word list CSV
```

## Notes

- The Google Sheet is sent with `no-cors`, so the page cannot confirm the row was actually written. The JSONL file holds the same scores and should be treated as the primary record.
- Only face numbers are shown on screen, but `target_word` is in the HTML source, so a participant who opens developer tools could see it.
