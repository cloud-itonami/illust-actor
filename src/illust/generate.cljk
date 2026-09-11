(ns illust.generate
  "Pure candidate builder for one co-scientist round (ADR-2607123000 §2/§3).

  Same 'closed hypothesis pool, no LLM in Generation' discipline
  cloud_murakumo.cosci uses: a small enumerable gene pool of
  subject/style/lighting variations, persona-flavored via `:persona/tags`.
  round-candidates is a pure function of (persona, round, k) — re-running the
  same round number reproduces the same candidates; exploration across the
  pool happens by round number advancing (illust.loop) and by biasing one
  gene slot toward the previous round's elite (illust.cosci/evolve-round)."
  (:require [kotoba.lang.text :as str]))

(def gene-pool
  {:subject ["a lone traveler at a rural train crossing"
             "a market stall under a striped awning"
             "a quiet harbor at dusk"
             "an overgrown shrine gate in a forest"
             "a night market food stand"
             "a mountain pass switchback road"
             "a riverside laundry line at sunrise"
             "a ferry terminal waiting room"]
   :style ["ghibli-inspired painterly" "flat cel-shaded" "watercolor sketch"
           "retro travel poster" "soft pastel gouache" "ukiyo-e woodblock"]
   :lighting ["golden hour" "overcast diffuse" "blue hour dusk"
              "midday clear" "warm interior lamplight"]})

(defn- pick [xs seed n] (nth xs (mod (+ seed n) (count xs))))

(defn- gene-for
  "One candidate's gene map. `bias` (from illust.cosci/evolve-round's elite,
  or nil on round 0) pins ONE randomly-chosen slot to the prior winner's
  value instead of round-robining it — elitism without literal crossover
  machinery, honest about being a small closed pool rather than a genuine
  genetic search."
  [round i bias]
  (let [raw {:subject  (pick (:subject gene-pool) round i)
             :style    (pick (:style gene-pool) round (+ i 1))
             :lighting (pick (:lighting gene-pool) round (+ i 2))}]
    (if (and bias (pos? round) (zero? (mod (+ round i) 3)))
      (merge raw (select-keys bias [(nth [:subject :style :lighting] (mod round 3))]))
      raw)))

(defn round-candidates
  "persona + round n (0-based) + k candidates + optional elite bias
  -> [{:candidate/id :prompt :gene :params} ...]."
  ([persona n k] (round-candidates persona n k nil))
  ([{:keys [tags]} n k bias]
   (vec
    (for [i (range k)]
      (let [{:keys [subject style lighting]} (gene-for n i bias)]
        {:candidate/id (str "r" n "-c" i)
         :prompt (str/join ", " (concat [subject style (str lighting " lighting")]
                                         tags))
         :gene {:subject subject :style style :lighting lighting}
         :params {}})))))
