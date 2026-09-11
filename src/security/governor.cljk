(ns security.governor
  "SecurityGuardGovernor — the independent safety/traceability layer
  named in this repository's README/business-model.md, gating every
  access-control override an advisor may propose for a site. The
  governor never dispatches hardware itself and never actuates
  use-of-force or detention. Modeled on cloud-itonami-isco-4311's
  bookkeeping.governor. Task twist: a proposed access level is an
  arithmetic ceiling against the site's registered access-control
  ceiling, and an access-control override cannot proceed until
  identity has been verified for that specific request.

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. client provenance   — the business/event organizer/residential
                             community must be registered.
    2. no-actuation        — proposal :effect must be :propose (the
                             governor never dispatches hardware and
                             never actuates use-of-force or
                             detention; it only gates what the
                             advisor may override).
    3. site basis          — an access-override proposal must cite a
                             REGISTERED site belonging to this
                             client.
    4. access-level ceiling — the proposed access level must not
                             exceed the site's registered
                             `:max-access-level` (granting access
                             beyond the site's registered level is an
                             unauthorized override, not efficient
                             service).
    5. identity verified   — the proposal must have
                             `:identity-verified?` true before any
                             access-control override can proceed (an
                             override without identity verification is
                             an unverified override, not efficient
                             service).
  ESCALATION invariants (:escalate? true, ALWAYS human sign-off per
  business-model.md's Trust Controls — these are :high/
  :safety-critical regardless of confidence):
    6. :op :approve-use-of-force-action (no use-of-force action
                             without the governor gate and human
                             sign-off).
    7. :op :approve-detention-action (no detention action without the
                             governor gate and human sign-off).
    8. low confidence (< `confidence-floor`)."
  (:require [security.store :as store]))

(def confidence-floor 0.6)

(def ^:private always-escalate-ops #{:approve-use-of-force-action
                                     :approve-detention-action})

(defn- hard-violations [{:keys [request proposal]} client-record s]
  (let [{:keys [op access-level identity-verified?]} proposal
        override? (= :approve-access-override op)]
    (cond-> []
      (nil? client-record)
      (conj {:rule :no-client :detail "未登録 client"})

      (not= :propose (:effect proposal))
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（governor は実力行使/拘束を直接実行しない）"})

      (and override? (nil? s))
      (conj {:rule :unknown-site :detail "未登録 site へのアクセス制御提案は不可"})

      (and override? s (not= (:client-id s) (:client-id request)))
      (conj {:rule :site-wrong-client :detail "site が別 client のもの"})

      (and override? s (number? access-level) (> access-level (:max-access-level s)))
      (conj {:rule :access-level-exceeds-limit
             :detail (str "アクセスレベル " access-level " > 登録済み上限 "
                          (:max-access-level s) "（登録上限を超えるアクセス許可は無許可オーバーライドであって効率的サービスではない）")})

      (and override? (not identity-verified?))
      (conj {:rule :identity-not-verified
             :detail "本人確認未完了のアクセス制御オーバーライドは未確認オーバーライドであって効率的サービスではない"}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `security.store/Store`. Pure — never mutates
  the store, never actuates use-of-force or detention."
  [request context proposal store]
  (let [client-record (store/client store (:client-id request))
        s (some->> (:site-id proposal) (store/site store))
        hard (hard-violations {:request request :proposal proposal}
                              client-record s)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        always-risky? (contains? always-escalate-ops (:op proposal))]
    {:ok? (and (not hard?) (not low?) (not always-risky?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky?))}))
