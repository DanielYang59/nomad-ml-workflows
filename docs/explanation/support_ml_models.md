# Supporting ML Models in NOMAD

This page outlines how ML models can be supported in NOMAD. The focus is on schema design, safe ingestion, provenance capture in NOMAD archives, and export to interoperable metadata formats.

## MLModel Schema

`MLModel` should be a NOMAD `Entity`. It represents the model artifact together with user-provided and computed metadata.

Core quantities:

- task (ML problem the model is meant to perform: classification, regression, segmentation, generation, etc.)
- description (something similar to Model Cards from HuggingFace)

Sub-sections:

- Training Data
  - internal references to NOMAD entries
  - file references to uploaded dataset files

- Metrics
  - training and testing metrics

- Architecture Details
  - input layer shape
  - output layer shape
  - model family or architecture name
  - architecture framework
  - tensor summary

- Optimization Details
  - optimizer name
  - optimization framework
  - key optimization settings

- Model Artifacts
  - path to model artifacts
  - artifact format (safetensors, pt, tf)
  - artifact checksum

### `results.material` Section Normalization

If the training data includes structured NOMAD entries, normalize the collective
material space into the `results.material` of model entry.

### Open Standards for MLModel Schema

No single open standard covers the full NOMAD scope for scientific ML models. A practical schema can combine the following standards.

- **[OpenMetadata `MlModel`](https://docs.open-metadata.org/v1.12.x/api-reference/sdk/python/entities/ml-model)**
  - useful for entity-level fields, model metadata, and lineage-friendly references
  - good starting point for identifiers, artifact locations, related data assets, and structured model properties
  - should be treated as a design reference, not as the source schema for NOMAD

- **[ML-Schema](https://ml-schema.github.io/documentation/)**
  - useful for the conceptual model of algorithms, tasks, datasets, runs, and trained models
  - helps define the scientific meaning of training, evaluation, and model generation steps
  - better suited to semantics than to direct NOMAD field design

- **[RO-Crate](https://www.researchobject.org/ro-crate/) and [W3C PROV](https://www.w3.org/TR/prov-overview/)**
  - useful for representing and packaging training provenance
  - good basis for mapping `archive.workflow2`, referenced inputs, produced artifacts, and exports
  - complements the `MLModel` schema rather than replacing it

- **[MLCommons Croissant](https://docs.mlcommons.org/croissant/docs/croissant-spec.html)**
  - useful for linked training-data metadata and export interoperability
  - good fit for dataset references associated with a model entry
  - not a full model schema and not a full provenance model

For NOMAD, the most natural approach is:

- use `MLModel` as a NOMAD `Entity`
- borrow entity structure from OpenMetadata where useful
- use ML-Schema as a semantic reference for model, task, and training concepts
- use RO-Crate and W3C PROV for provenance packaging
- use Croissant for linked dataset export

## Ingestion/Parsing Support

Model ingestion is one part of ML model support in NOMAD. Version 1 supports automatic parsing only for `safetensors` files.

- `safetensors` files are parsed automatically and generate an `MLModel` entry
- other model formats (.pt, .tf, etc.) are not auto-loaded
- non-`safetensors` models require manual entry creation by the user

### Scope of safetensors Parsing

Automatically populated:

- Model artifacts path, format, checksum

Conditionally populated from header metadata:

- Architecture details: framework, model family or architecture name

## Training Provenance in NOMAD

Training provenance is stored with current NOMAD workflow fields. Creating a separate entry for the Training activity might be reasonable here.

- inputs: referenced training-data entries and optional config or artifact entries
- output: the `MLModel` entry
- without such references, no training provenance graph is auto-created

## Export

### HF ModelCard Export

An `MLModel` entry can also be exported as a Hugging Face ModelCard. This is useful for model sharing, human-readable documentation, and interoperability with the Hugging Face Hub.

- export selected `MLModel` metadata into a ModelCard structure
- use it as an optional sharing export, not as the source schema
- treat it as a lightweight and potentially lossy projection of the NOMAD entry

### Croissant Export

Exporting a model entry should also produce a Croissant file for interoperability. The export covers model metadata, linked training-data references, and exported artifact references. Croissant is not treated as the full provenance representation.

### RO-Crate for Provenance

RO-Crate is being explored as a richer packaging format for model training provenance. It can bundle model entry metadata, workflow provenance, referenced training data, and associated artifacts or exports.

## Current Boundaries

- only `safetensors` gets automatic parsing
- provenance depends on explicit entry references
