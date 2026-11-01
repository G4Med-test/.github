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

> [!IMPORTANT]
> **You cannot push to `main`.** The validation runs on INFN self-hosted runners, so no
> code reaches them without review. Make every change on a branch and open a pull
> request.

1. **Create the repository**: *[Use this template](https://github.com/G4Med-test/template/generate)*,
   owner `G4Med-test`. The repository name becomes the name of the executable and of the
   container. Then create a branch for your work, e.g. `git switch -c my-change`.
2. **Write the application** in `main.cc`, `src/` and `include/`.
3. **Add the macros** to `macro/`: `unit.mac` is a one-event smoke test run on every
   push; the validation macros can have any name and must be self-contained.
   Put **every input parameter in the macro** (geometry, physics, beam, seed), as UI
   commands, rather than in command-line arguments or environment variables: the
   pipeline runs only `<executable> <macro>`, and a macro that describes the whole run
   keeps the test simple and its results reproducible. To scan a parameter, write one
   macro per value. See the
   [template README](https://github.com/G4Med-test/template#put-every-input-parameter-in-the-macro).
4. **List the validation macros** in `validation/config.json`, e.g.
   `{"macros": ["proton_100MeV.mac"]}`.
5. **Write the parser** in `validation/parser.py`: `metadata(commands)` describes the run
   from its macro, `parse(job)` reads the output files and returns the plots for the
   validation portal.
6. **Push your branch and open a pull request** to `main`:
   - the build and the unit test run immediately on GitHub runners;
   - a maintainer reviews the changes and approves them;
   - only then do the validation macros run on the INFN runners, and the results are
     collected.

   You can follow each step on the
   [dashboard](https://g4med-test.github.io/dashboard/).
7. **When the test is ready**, add it to
   [`ci-workflows/tests.json`](https://github.com/G4Med-test/ci-workflows/blob/main/tests.json)
   so that it runs with every new Geant4 version.

The [template README](https://github.com/G4Med-test/template#readme) describes the
parser interface and how to build and run each step by hand.

## Running the tests on your machine or on a farm

The pipeline also runs outside GitHub, on any Linux machine with
[Apptainer](https://apptainer.org) ≥ 1.5, bash and git: a workstation, or the nodes of a
Slurm or HTCondor farm. One script of `ci-workflows` does what the CI does, for one test
or for all of them: the same image, the same macros run in the same way, the same results.

```bash
git clone https://github.com/G4Med-test/ci-workflows.git
P=ci-workflows/scripts/local-pipeline.sh     # $P --help describes every command
```

The Geant4 datasets come from CVMFS if it is mounted; otherwise they are downloaded
once per Geant4 version (about 2 GB). On a farm, `--dir` must be on a filesystem shared
with the nodes. The
[ci-workflows README](https://github.com/G4Med-test/ci-workflows#running-the-pipeline-outside-github-actions)
has the details.

### While developing a test

Push your branch. The CI builds its image and the script downloads it, so you do not
need permission to build containers. Then run the whole test, validation macros
included, without waiting for the review or the INFN runners:

```bash
$P prepare --dir ~/g4med --ref my-change MyTest   # image of the branch, datasets, macro list
$P unit    --dir ~/g4med MyTest                   # the unit macro, as on every push
$P submit  --dir ~/g4med local -j 4               # every validation macro, 4 at a time, then the plots
$P status  --dir ~/g4med
```

- `~/g4med/MyTest/runs/<id>/outputs/`: output files and logs of each macro;
- `~/g4med/MyTest/validation-json/`: the plots for the portal, as the CI would publish them.

After pushing new commits, run `prepare` again once their build on GitHub has finished
(the *Build* stage on the dashboard): the next runs use the new image. For a
pull request from a fork, whose image the CI does not publish, add `--build` to build
the image on your machine (this needs `apptainer build`, for example
`APPTAINER_BUILD_FLAGS=--fakeroot`). To try changes you have not committed yet, follow
the step-by-step commands of the template README.

Results of a branch are for checking your work: they cannot be published.

### Contributing results

If you have computing resources, you can run the validation of the tests on them and
propose the results to `validation-results`:

```bash
$P prepare --dir /shared/g4med all                        # or name the tests; --geant4 v11.4.3 for another version
$P submit  --dir /shared/g4med slurm -- --partition=long  # or: condor -- ..., or local -j N
$P status  --dir /shared/g4med
$P collect --dir /shared/g4med                            # with condor only: local and slurm collect by themselves
$P publish --dir /shared/g4med
```

`publish` writes the results with the layout of the CI on the branch
`results/<test>-local-<user>` and opens a pull request. A maintainer reviews and merges
it; nothing is merged automatically. On the dashboard these results are marked
*Local run*, with the user and the machine that ran them.

Only results the CI could have produced are published, so they are as reproducible as
the CI's:

- the test runs from a commit of its `main` branch, with no local changes;
- the image is the one the CI built for that commit. For a Geant4 version that the CI
  has not built for a test yet, ask a maintainer to start the validation of that version
  from the [dashboard](https://g4med-test.github.io/dashboard/geant4.html).

Publishing needs write access to `validation-results` (ask a maintainer); use
`G4MED_GIT_URL=git@github.com:G4Med-test` to push with SSH. If the
[GitHub CLI](https://cli.github.com) is installed, `publish` opens the pull request;
otherwise it prints the link.
