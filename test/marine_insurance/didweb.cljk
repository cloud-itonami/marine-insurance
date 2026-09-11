(ns marine_insurance.didweb
  "did:web の解決規則だけを持つ。**URL を手で書かないため**にある。

   手書きの URL を測ると『存在しない URL を測って 404 だと報告する』ことになり、
   実測しているつもりで自分の打ち間違いを測る。DID から規則で導出すれば、
   測っている先が名乗りと一致していることが構造的に保証される。

   W3C did:web:
     did:web:HOST         -> https://HOST/.well-known/did.json
     did:web:HOST:a:b     -> https://HOST/a/b/did.json"
  (:require [kotoba.lang.text :as str]))

(defn did->url
  [did]
  (when (str/starts-with? (str did) "did:web:")
    (let [segs (-> (subs (str did) (count "did:web:"))
                   (str/split #":"))
          host (first segs)
          path (rest segs)]
      (when (seq host)
        (if (seq path)
          (str "https://" host "/" (str/join "/" path) "/did.json")
          (str "https://" host "/.well-known/did.json"))))))

(defn did->host
  [did]
  (when-let [u (did->url did)]
    (-> u (str/replace #"^https://" "") (str/split #"/") first)))
