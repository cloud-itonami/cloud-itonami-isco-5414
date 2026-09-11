(ns security.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [security.store :as store]
            [security.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-client! st {:client-id "client-1" :name "Kobo Security"})
    (store/register-site! st {:site-id "S-1" :client-id "client-1"
                              :name "site-042"
                              :max-access-level 3})
    st))

(defn- override-op [level verified?]
  {:op :approve-access-override :effect :propose :site-id "S-1"
   :access-level level :identity-verified? verified?
   :confidence 0.9 :stake :low})

(def ^:private req {:client-id "client-1"})

(deftest ok-within-level-and-verified
  (let [st (fresh-store)
        v (governor/check req {} (override-op 2 true) st)]
    (is (:ok? v))))

(deftest ok-at-exact-level-boundary
  (testing "the access-level ceiling is inclusive"
    (let [st (fresh-store)
          v (governor/check req {} (override-op 3 true) st)]
      (is (:ok? v)))))

(deftest hard-on-access-level-exceeds-limit
  (testing "granting access beyond the site's registered access level is an unauthorized override, not efficient service"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (override-op 9 true) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :access-level-exceeds-limit (:rule %)) (:violations v))))))

(deftest hard-on-identity-not-verified
  (testing "an access-control override without identity verification is an unverified override, not efficient service"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (override-op 2 false) :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :identity-not-verified (:rule %)) (:violations v))))))

(deftest hard-on-unknown-site
  (let [st (fresh-store)
        v (governor/check req {} (assoc (override-op 2 true) :site-id "S-ghost") st)]
    (is (:hard? v))
    (is (some #(= :unknown-site (:rule %)) (:violations v)))))

(deftest hard-on-foreign-site
  (let [st (fresh-store)]
    (store/register-client! st {:client-id "client-2" :name "Other"})
    (let [v (governor/check {:client-id "client-2"} {} (override-op 2 true) st)]
      (is (:hard? v))
      (is (some #(= :site-wrong-client (:rule %)) (:violations v))))))

(deftest hard-on-unregistered-client
  (let [st (fresh-store)
        v (governor/check {:client-id "nobody"} {} (override-op 2 true) st)]
    (is (:hard? v))
    (is (some #(= :no-client (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        v (governor/check req {} (assoc (override-op 2 true) :effect :direct-write) st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest always-escalates-use-of-force-action-even-at-high-confidence
  (testing "no use-of-force action without the governor gate and human sign-off"
    (let [st (fresh-store)
          v (governor/check req {} {:op :approve-use-of-force-action :effect :propose
                                    :site-id "S-1" :confidence 0.99 :stake :low} st)]
      (is (not (:hard? v)))
      (is (:escalate? v)))))

(deftest always-escalates-detention-action-even-at-high-confidence
  (testing "no detention action without the governor gate and human sign-off"
    (let [st (fresh-store)
          v (governor/check req {} {:op :approve-detention-action :effect :propose
                                    :site-id "S-1" :confidence 0.99 :stake :low} st)]
      (is (not (:hard? v)))
      (is (:escalate? v)))))

(deftest escalates-low-confidence
  (let [st (fresh-store)
        v (governor/check req {} (assoc (override-op 2 true) :confidence 0.3) st)]
    (is (not (:hard? v)))
    (is (:escalate? v))))
