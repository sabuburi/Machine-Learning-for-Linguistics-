# PROJECT BRIEF: Muallim vs Mujawwad recitation (ML for Linguistics final report)

Read this whole file before doing anything. Items marked **TO CONFIRM** are not settled.
Do not guess them: ask me.

## 1. Goal
A final report for a *Machine Learning for Linguistics* course.
Working title: **"Vowel duration and vowel space in Muallim vs Mujawwad Quranic recitation (word-level audio)"**
Core question: for the same Arabic words, do the two recitation styles differ in
(a) vowel duration, (b) vowel quality (F1/F2 vowel space), and can a classifier
(XGBoost) tell the styles apart from these measurements?

The framing is **descriptive phonetics**. No ranking of styles or reciters, no religious judgments.

## 2. Professor's suggestions (my notes, verbatim) and how I read them
Verbatim notes:
- Vowel Duration
- Check class three, Euclidean Distance
- Look at the Praat analysis
- Run an xG boost
- Vowel Identity, F_one vs F_one
- Dependent variable, Teacher vs Student

| Note | My reading | Status |
|---|---|---|
| Vowel duration | Measure duration of each vowel (short /a i u/ vs long /aː iː uː/), compare across styles | clear |
| Check class three, Euclidean Distance | Most likely **k-means clustering** of vowel tokens on standardized F1, F2, and duration (k-means groups points by Euclidean distance). Reference R script: `docs/class3/kmeans_formants.R` (scale first, k = 6 for the six Arabic vowels, nstart = 100, fixed seed). Also useful: Euclidean distance between cluster/vowel centroids per style. | likely, **TO CONFIRM** that this is the class-3 method |
| Look at the Praat analysis | Use Praat (or its Python wrapper `praat-parselmouth`) for formants, durations, and TextGrids | clear |
| Run an xG boost | XGBoost classifier, grouped cross-validation, baselines | clear |
| Vowel Identity, F_one vs F_one | Almost certainly **F1 vs F2** (typo): vowel-space plot per vowel identity | **TO CONFIRM** (typo) |
| Dependent variable, Teacher vs Student | Correction from me: the dependent variable is **style = Muallim vs Mujawwad**. "Teacher vs Student" in the notes was a slip. | confirmed |

Dependent variable (confirmed): a column `style` with values `muallim` / `mujawwad`,
derived from the top-level folder name. There is no teacher/student label anywhere in this project.

## 3. Data
- Hugging Face dataset `zaibihassan/Quranic-Word-By-Word-Audio-Data` (Apache-2.0).
- Layout: `muallim/<SSS>/<SSS>_<AAA>_<WWW>.opus`, `mujawwad/<SSS>/...`, plus one `.pb`
  timestamp file per surah (schema unknown). SSS = surah, AAA = ayah, WWW = word index.
- Audio: 16 kHz mono Opus, converted from MP3 with the same pipeline for both styles.
- ~77k words per style, all 114 surahs. Local subset: `data/quran_wbw/`, surahs 78-114.
- The README does **not name the reciter(s)**. Unknown whether it is the same voice in both styles.
- The Hugging Face viewer's `label` column appears to be the surah folder, NOT the style.
  Always derive style from the top-level folder name.
- The dataset has **no text**. To know vowel identity we need the diacritized Quran text
  (e.g. Tanzil Uthmani) mapped to SURAH_AYAH_WORD. **Validate the mapping** by comparing
  the number of words per ayah in the text with the number of audio files per ayah.
  Report any mismatches; do not silently drop or realign.
- There are no vowel boundaries in the data. Vowel segmentation is the main technical problem.

## 4. Existing code
`style_classifier.py` works at word level on synthetic test data only. It has NOT been run on
the real files. It extracts duration, pause, RMS, F0 (semitones), MFCC features, and runs
surah-grouped CV with majority, duration-only, logistic regression, and random forest models.
It needs: XGBoost, Praat/parselmouth formant measurements, and vowel-level features.

## 5. Methodological rules
- Matched design: compare the same word (same SSS_AAA_WWW) across styles.
- Cross-validation grouped by surah. No clip, word, or vowel from one surah in both train and test.
- Baselines: majority class; duration-only. Ablation: model without duration.
- Formant settings are parameters (max formant 5000 for male, 5500 for female, 5 formants,
  25 ms window). Check them on samples. Manually inspect at least 50 random tokens
  (spectrogram + formant tracks) and report the error rate.
- Report every exclusion (too short, no voicing, failed formant tracking) with counts.
- Fixed random seeds; every figure and table reproducible from one script.
- Treat Euclidean distance computations as undefined until the class-3 definition is confirmed.

## 6. Known limitations to state in the report
- Reciters unknown; style may be confounded with voice.
- Opus compression at 16 kHz may slightly affect formant estimates.
- Recitation is a codified register (tajweed, madd elongation), not natural speech.
- Word clips remove inter-word pauses; vowel boundaries are estimated, not given.

