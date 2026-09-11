(ns marine_insurance.murakumo
  "Pure cljc actor boundary generated from manifest migration scaffold."
  (:require [kotoba.lang.text :as str]))

(def actor-did
  "did:web:marine-insurance.etzhayyim.com")

(def common-gates
  [:council-charter-attestation
   :no-platform-held-key-baseline
   :no-probing-baseline
   :murakumo-only-inference-baseline
   :did-primary-baseline
   :append-only-gate-baseline
   :kotoba-only-substrate-baseline])

(defn collection
  [name]
  (str "com.etzhayyim.marine-insurance." name))

(def cell-specs {
  :getpolicy {:legacy-cell "com-etzhayyim-apps-marineInsurance-underwriting-getPolicy"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "getpolicy")]
     :required-gates common-gates
     :trigger "manifest cell getpolicy"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :listpolicies {:legacy-cell "com-etzhayyim-apps-marineInsurance-underwriting-listPolicies"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "listpolicies")]
     :required-gates common-gates
     :trigger "manifest cell listpolicies"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :listclaims {:legacy-cell "com-etzhayyim-apps-marineInsurance-underwriting-listClaims"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "listclaims")]
     :required-gates common-gates
     :trigger "manifest cell listclaims"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :listpientries {:legacy-cell "com-etzhayyim-apps-marineInsurance-underwriting-listPiEntries"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "listpientries")]
     :required-gates common-gates
     :trigger "manifest cell listpientries"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :getvesselcoverage {:legacy-cell "com-etzhayyim-apps-marineInsurance-underwriting-getVesselCoverage"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "getvesselcoverage")]
     :required-gates common-gates
     :trigger "manifest cell getvesselcoverage"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :health {:legacy-cell "com-etzhayyim-apps-marineInsurance-health"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "health")]
     :required-gates common-gates
     :trigger "manifest cell health"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :get {:legacy-cell "com-etzhayyim-apps-marineinsurance-coverage-get"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "get")]
     :required-gates common-gates
     :trigger "manifest cell get"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :shinkaevolution {:legacy-cell "com-etzhayyim-apps-standard-shinkaEvolution"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "shinkaevolution")]
     :required-gates common-gates
     :trigger "manifest cell shinkaevolution"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :shinkaknowledge {:legacy-cell "com-etzhayyim-apps-standard-shinkaKnowledge"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "shinkaknowledge")]
     :required-gates common-gates
     :trigger "manifest cell shinkaknowledge"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :shinka {:legacy-cell "shinka"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "shinka")]
     :required-gates common-gates
     :trigger "manifest cell shinka"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :koji {:legacy-cell "koji"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "koji")]
     :required-gates common-gates
     :trigger "manifest cell koji"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :kyumei {:legacy-cell "kyumei"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "kyumei")]
     :required-gates common-gates
     :trigger "manifest cell kyumei"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :domain-knowledge {:legacy-cell "domain-knowledge"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "domain-knowledge")]
     :required-gates common-gates
     :trigger "manifest cell domain-knowledge"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :policy {:legacy-cell "com-etzhayyim-apps-marineInsurance-policy"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "policy")]
     :required-gates common-gates
     :trigger "manifest cell policy"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :claim {:legacy-cell "com-etzhayyim-apps-marineInsurance-claim"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "claim")]
     :required-gates common-gates
     :trigger "manifest cell claim"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :generalaverage {:legacy-cell "com-etzhayyim-apps-marineInsurance-generalAverage"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "generalaverage")]
     :required-gates common-gates
     :trigger "manifest cell generalaverage"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :pientry {:legacy-cell "com-etzhayyim-apps-marineInsurance-piEntry"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "pientry")]
     :required-gates common-gates
     :trigger "manifest cell pientry"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
  :ship {:legacy-cell "com-etzhayyim-apps-vessel-ship"
     :phase :event
     :murakumo-node "reuben"
     :collections [(collection "ship")]
     :required-gates common-gates
     :trigger "manifest cell ship"
     :ceiling "Manifest-driven migration scaffold; explicit execution stays in runtime methods"}
})

(defn safe-rkey
  [s]
  (let [clean (-> (str s)
                  (str/replace #"^did:web:" "")
                  (str/replace #"[^A-Za-z0-9._~-]" "-"))]
    (if (str/blank? clean) "unknown" clean)))

(defn gate-value
  [attestations gate]
  (or (get attestations gate)
      (get attestations (name gate))
      (when (set? attestations) (attestations gate))
      (when (set? attestations) (attestations (name gate)))))

(defn missing-gates
  [spec attestations]
  (->> (:required-gates spec)
       (remove #(boolean (gate-value attestations %)))
       vec))

(defn put-record-effect
  [collection rkey record]
  {:op :mst/put-record
   :actor actor-did
   :collection collection
   :rkey rkey
   :record record})

(defn records-for
  [spec {:keys [records record computed-at request-id]
         :as input}]
  (let [input-records (cond
                        (map? records) records
                        (some? record) {0 record}
                        :else {})
        base {:actorDid actor-did
              :computedAt computed-at
              :legacyCell (:legacy-cell spec)
              :phase (:phase spec)
              :requestId request-id
              :actorBoundary "cljc-migration-scaffold"
              :scaffold true
              :constitutionalStatus "attested-plan"}]
    (map-indexed
     (fn [idx coll]
       (let [record* (merge {:$type coll}
                            base
                            (or (get input-records coll)
                                (get input-records idx)
                                {}))
             rkey (safe-rkey (or (:rkey record*)
                                 (get record* "rkey")
                                 (:tid record*)
                                 request-id
                                 (str (:legacy-cell spec) "-" idx)))]
         {:collection coll
          :record record*
          :rkey rkey}))
     (:collections spec))))

(defn cell-plan
  [cell-key {:keys [attestations] :as input}]
  (let [spec (get cell-specs cell-key)]
    (when-not spec
      (throw (ex-info "unknown cell" {:cell cell-key})))
    (let [missing (missing-gates spec attestations)]
      (merge
       {:cell cell-key
        :legacy-cell (:legacy-cell spec)
        :actor actor-did
        :phase (:phase spec)
        :murakumo-node (:murakumo-node spec)
        :trigger (:trigger spec)
        :ceiling (:ceiling spec)
        :required-gates (:required-gates spec)
        :missing-gates missing}
       (if (seq missing)
         {:status :blocked
          :effects []}
         (let [planned-records (records-for spec input)]
           {:status :ready
            :records (vec planned-records)
            :effects (mapv (fn [{:keys [collection record rkey]}]
                             (put-record-effect collection rkey record))
                           planned-records)}))))))

(defn all-cell-plans
  [input]
  (into {}
        (map (fn [cell-key] [cell-key (cell-plan cell-key input)]))
        (keys cell-specs)))
