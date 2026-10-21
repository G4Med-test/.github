# G4-Med: Geant4 validation for medical physics

This organization collects the tests that validate the physics models of
[Geant4](https://geant4.web.cern.ch) used in medical physics, such as radiotherapy,
hadrontherapy, brachytherapy, imaging, dosimetry and microdosimetry.
It is maintained by the Geant4 Medical Simulation Benchmarking Group.

Each repository is one test: a Geant4 application that reproduces a reference
measurement or calculation, for example stopping powers, attenuation coefficients,
electron backscattering, Bragg peaks, nuclear fragmentation or linac dose
distributions. Every new Geant4 release runs all the tests automatically, so changes in
the agreement with the reference data show up as soon as the release is published.
The results are published in
[`validation-results`](https://github.com/G4Med-test/validation-results).

📊 **[Dashboard](https://g4med-test.github.io/dashboard/)**: status of every test,
latest runs, Geant4 versions and open pull requests.

## Publications

The benchmarking system and its results are described in:

1. P. Arce *et al.*, "Report on G4-Med, a Geant4 benchmarking system for medical
   physics applications developed by the Geant4 Medical Simulation Benchmarking Group",
   *Medical Physics* **48**(1), 19–56 (2021).
   doi:[10.1002/mp.14226](https://doi.org/10.1002/mp.14226)
2. P. Arce *et al.*, "Results of a Geant4 benchmarking study for bio-medical
   applications, performed with the G4-Med system",
   *Medical Physics* **52**(5), 2707–2761 (2025).
   doi:[10.1002/mp.17678](https://doi.org/10.1002/mp.17678)

Please cite them if you use these tests or their results.

## For test developers

A test is a repository created from
[`template`](https://github.com/G4Med-test/template). All of them share the pipeline in
[`ci-workflows`](https://github.com/G4Med-test/ci-workflows), which runs in the
[`geant4-alma9`](https://github.com/G4Med-test/geant4-alma9) Apptainer image.

1. **Create the repository**: *[Use this template](https://github.com/G4Med-test/template/generate)*,
   owner `G4Med-test`. The repository name becomes the name of the executable and of the
   container.
2. **Write the application** in `main.cc`, `src/` and `include/`.
3. **Add the macros** to `macro/`: `unit.mac` is a one-event smoke test run on every
   push; the validation macros can have any name and must be self-contained.
4. **List the validation macros** in `validation/config.json`, e.g.
   `{"macros": ["proton_100MeV.mac"]}`.
5. **Write the parser** in `validation/parser.py`: `metadata(commands)` describes the run
   from its macro, `parse(job)` reads the output files and returns the plots for the
   validation portal.
6. **Open a pull request.** The build and the unit test run on GitHub; after a maintainer
   approves, the validation macros run on the Padova runners and the results are
   collected. You can follow each step on the
   [dashboard](https://g4med-test.github.io/dashboard/).
7. **When the test is ready**, add it to
   [`ci-workflows/tests.json`](https://github.com/G4Med-test/ci-workflows/blob/main/tests.json)
   so that it runs with every new Geant4 version.

Everything can also be built and run locally with Apptainer: the
[template README](https://github.com/G4Med-test/template#readme) has the details, the
commands and the description of the parser interface.
