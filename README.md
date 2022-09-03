# steady-vs-drifting-features
Ideas about ML models feature types and stages 

## Steady/unfitted features
Features whose concept does not change over time. For example a *maximum number of daily orders in the past 30 days*. The number changes over time (or more generally, with different input data), but the feature's concept (what it is supposed to represent) stays the same.

## Drifting/fitted features
Features whose concept is sensitive to data/concept drift. For example *normalized monthly income of a customer*. Not only does the output value change with input values, but the feature's concept does also. This is because the transformation that produces the feature is fitted on some past (training) dataset. In a sense it is a primitive model in itself which "learns" the mean and the variance of that past dataset. The feature is supposed to represent a **normalized** monthly income, but this is only true as long as the data underlying distribution doesn't change (or at least its first 2 moments).

More precisely, a feature F is steady iff F(D_fit1, D_inf) == F(D_fit2, D_inf) for all D_fit1, D_fit2 \in {valid datasets} and all D_inf \in {valid datasets}. In other words, F is fitted only in the implicit sense that it might depend on the definition of a validt dataset.

## Why should I care?
In desigining data processing and ML pipelines and particularly when designing a feature store, is is important to realize what components should or should not be coupled. On one hand, it is possible to create a model service which consumses nothing but "raw" data as inputs and which has all data transformations "baked into itself". The transformations are then tightly coupled with the model. On the other extreme, one can design a pattern where all data are processed by a separate data transformation service and the ML model itself consists only of a final estimator (i.e. all pre-processing is done separately).

I would argue that the distinction between steady and drifting features illustrates that the latter approach is (usually) an antipattern. ML models are subject to data drift, concept drift etc and so are the drifting features as defined above. In order to prevent a model from deterioriating in performance, it is important to monitor and retrain it. By keeping the drifting features tightly coupled with the model itself, we ensure that we cover the drift in the preprocessing as well. Otherwise, we need to a) monitor these features separately and b) couple this monitoring (and possible refitting) with monitoring and training pipelines of all models that consume these features. This might be justifiable in some scenarios (e.g. when the fitted features are very complex, are themselves products of separate ML algorithms or are consumed by other services such as BI), but should be avoided otherwise.

On the other hand, it makes a lot of sense to decouple the generation of steady features from a specific model, at least as long as we (expect to) have more models in production. This comes with several benefits:

1. Reusability. The feature only needs to be implemented once and then consumed by various (not necessarily ML) processes. 
2. A single source of truth. We don't have to worry about having conflicting implementations of the same conceptual feature across different models.
3. Losser coupling/dependencies. We might be using several different frameworks or programming languages for our ML models (or other consumers of the feauture) or we might plan to do so. This way, we don't have to rewrite our feature each time our model framework changes (or write and keep consistent versions of the feature in different frameworks).
