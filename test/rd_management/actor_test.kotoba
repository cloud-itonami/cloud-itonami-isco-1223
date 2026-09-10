(ns rd-management.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [rd-management.actor :as actor]
            [rd-management.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-project! st {:project-id "project-1" :name "Battery R&D"})
    st))

(deftest commits-a-clean-low-risk-request
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:project-id "project-1" :op :coordinate :stake :low}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "project-1"))))))

(deftest holds-on-unregistered-project-without-committing
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:project-id "no-such-project" :op :coordinate :stake :low}
        result (actor/run-request! graph request {} "thread-2")]
    (is (= :done (:status result)))
    (is (nil? (get-in result [:state :record])))
    (is (empty? (store/records-of st "no-such-project")))
    (is (= :hold (:disposition (:state result))))))

(deftest interrupts-then-commits-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        ;; hazmat protocol approval always escalates (governor invariant)
        request {:project-id "project-1" :op :approve-hazmat-protocol :stake :high}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "project-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (some? (get-in resumed [:state :record])))
      (is (= 1 (count (store/records-of st "project-1")))))))
