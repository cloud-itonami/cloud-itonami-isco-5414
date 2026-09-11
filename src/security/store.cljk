(ns security.store
  "SSoT for the ISCO-08 5414 independent security guard practice
  actor (itonami actor pattern, ADR-2607011000 / CLAUDE.md Actors
  section; README's 'Robotics premise' — a patrol-support and
  access-log robot performs perimeter patrol logging, badge scanning
  and incident-report printing under this advisor/governor pair,
  which never dispatches hardware itself and never actuates
  use-of-force or detention). Modeled on cloud-itonami-isco-4311's
  bookkeeping.store.

  Domain:

    client — a registered business/event organizer/residential
             community (:client-id, :name)
    site   — a registered patrol site {:site-id :client-id :name
             :max-access-level number}. `:max-access-level` is the
             registered access-control ceiling a proposed access-
             control override's access level must not exceed —
             granting access beyond the site's registered access
             level is an unauthorized override, not efficient
             service.
    record — a committed operating record (a logged access-control
             override) — written ONLY via commit-record!.
    ledger — append-only audit trail, commit or hold."
  )

(defprotocol Store
  (client [s client-id])
  (site [s site-id])
  (records-of [s client-id])
  (ledger [s])
  (register-client! [s client])
  (register-site! [s site-rec])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (client [_ client-id] (get-in @a [:clients client-id]))
  (site [_ site-id] (get-in @a [:sites site-id]))
  (records-of [_ client-id] (filter #(= client-id (:client-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-client! [s client]
    (swap! a assoc-in [:clients (:client-id client)] client) s)
  (register-site! [s site-rec]
    (swap! a assoc-in [:sites (:site-id site-rec)] site-rec) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:clients {} :sites {} :records [] :ledger []}
                                   seed)))))
