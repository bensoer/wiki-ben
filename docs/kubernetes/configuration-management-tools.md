# Configuration Management Tools
The following doc is a breakdown of some of the different configuration management tools out there for kubernetes, along with comments and considerations on their uses

In summary, the following are some known tools that exist:

| Name | Description | Links |
| ---- | ----------- | ----- |
| `helm` | | |
| `kustomize` | | |
| `jsonnet` / `ksonnet` | Builds on the shortcomings of YAML/JSON (used in Helm and Kustomize) where there is no programmable logic such as reusable variables | |
| `kapitan` | | |
| `cdk8s` | Developed by AWS, allows you to generate manifests using software languages | |
| `cuelang` | | |

The main debate is always, whats the difference between them all ? And which one should you use ? This varies on your use case I think.

## Helm vs Kustomize
These two are the most commonly debated between and they both approach manifest management in slightly different ways. But they also come with their limitations. For small / medium scale projects, either of these will probably do.

A major defining difference between the two is Helm keeps template control within the package itself, so the end user can only modify it from the available `values.yaml`. Kustomize gives the end user full control as they can write overlays for absolutely everything within it. This functionality is most prominent when you have hierarachies of manifests or multiple use-cases being needed for a single application

In Helm, all use-cases (whatever your Helm chart is designed to do) have to be accomodated within the template files and be accessible / configurable with configuration. In Kustomize, this is not an issue, as the end user can modify the Manifests as they wish. Kustomise, though means that the end user _must_ define everything they want to override, and do so correctly to still work with the original application. 

Basically, when you are using Helm charts, developed by others, you do not have to worry about what configurations work, because they will not be available to you, BUT you are limited only to what has been defined as possible by the Helm chart. When using Kustomize, you have the freedom to add and develop and configure whatever you want ontop of the original Kustomize, but it is then up to you to make sure the application still works.

## Why use CUE / cuelang
DevOps Toolkit does a very good breakdown of the requirements and reasons to choose cuelang: [https://www.youtube.com/watch?app=desktop&v=m6g0aWggdUQ&t=0s](https://www.youtube.com/watch?app=desktop&v=m6g0aWggdUQ&t=0s)

cuelang can also work to provide a missing feature in Helm - the ability to have variables in the `values.yaml`. This provides more dynamic definitions and saves typing writing configuration over and over as the `values.yaml` is just yaml, and doesn't offer any common variables, conditions or ability to share logic. cuelang can be used to write your entire application configurations, but if you already are setup with Helm, you can use it to fill in this known shortcoming of Helm, and migrate over eventually as you go.

One of the major points DevOps Toolkit makes of using cuelang over Helm and Kustomize, is that both of these systems are text template parsers, they don't work with YAML, they don't recognise its YAML underneath, and thus also _do not_ validate the YAML produced from themselves. These means you can have highly organised incorrect and invalid YAML. Which is pretty easy to become a debugging nightmare.

CUE / cuelang is a language that was created without any influence or intention of K8s. There are a few tools that have evolved cuelang further for use with kubernetes:

* Timoni
* KCL