{:layout :post, :title "Porting blog to cryogen", :date "2020-07-26" :draft? false}

Given I'm really digging clojure at the moment, I thought I'd port my blog over from octopress to a clojure based static site generator. I wanted something that supported the same sort of structure, with markdown, and low hassle on getting it running. [Cryogen](https://cryogenweb.org/) seemed to be the best documented and most modern of the few I could find.

The process to port it over was as follows
- Setup the right values in `content/config.edn`
- Port some content
- Setup deployment

## Config

The [config file](https://github.com/rickerbh/blog/blob/0ab0bd8250ab78a31d75db501892cddb9b13a7a6/content/config.edn) was trivial. The only bits that needed tweaking were the blog-prefix (I want all content at the root so `/` is right for me), `page-root-uri` (needed to make it empty string so pages are at existing URLs), and the `post-root-uri`. The last is a bit trickier - I couldn't find a way to support the existing post structure (wordpress style, with `/yyyy/MM/dd/post-name`), but that doesn't worry me too much. Also needed to add my custom historic uploads directory in to `:resources` array.

## Port the content

For content porting, this was a bit trickier. Cryogen requires front-matter to be in a clojure map, but octopress supported the frontmatter as yaml. So, I wrote a short (it was short, until I needed to deal with the edge cases) [babashka](https://github.com/borkdude/babashka/) script to do this. The script will take a directory of posts (including subdirs), read, and reformat them for cryogen. Basically they port the frontmatter into clojure maps, and filter out unsupported keys.

Script available at: [octopress-to-cryogen.clj](https://gist.github.com/rickerbh/4bcc3b3243e0f271b9ca3c8e16028948)

## Deployment

I was using github-pages to serve the existing site. The deployment process I had was a hassle. When I set it up I wrote a custom script that I'd need to run on my local machine to build then deploy the site to the special repo/branch to get github to deploy it. I'd always forget what the order of commands were, how I needed to init the ruby environment (did I use dotenv, or bundle, or both, or something else?), it was always a hassle.

I've used a number of different CI/automation systems (Jenkins, Travis, and CircleCI), but wanted to use this as an opportunity to learn a bit about GitHub Actions. I needed the configuration script to handle building a clojure project, and then deploying it to gh-pages. This required plugging together a few actions, but I didn't need to write any custom code for this. Everything I needed already existed, and needed to be coupled together.

The setup for the clojure part of the project just required following instructions at [https://github.com/marketplace/actions/setup-clojure](https://github.com/marketplace/actions/setup-clojure), and the publishing to gh-pages part at [https://github.com/marketplace/actions/github-pages-action](https://github.com/marketplace/actions/github-pages-action). The trickiest part was determining that because I'm publishing a "user" based page, I needed to use the `personal_token` and `external_repository` config.

GitHub Actions deployment script available at [workflow.yml](https://github.com/rickerbh/blog/blob/0ab0bd8250ab78a31d75db501892cddb9b13a7a6/.github/workflows/workflow.yml)

## End

I _do_ want to blog more - I enjoy it, and fingers crossed this simpler setup helps me do so.

Hopefully this helps you configure up cryogen for automated deployment via github actions, or the script to port content is useful!
