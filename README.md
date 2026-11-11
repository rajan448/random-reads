# Explainer Books

A Markdown book library, published with Jekyll on GitHub Pages. The home page automatically links to every `explainer-books/<folder>/00-overview.md` file. Existing chapter links work both on GitHub and on the published site.

## Publish on GitHub Pages

1. Create a GitHub repository and push this directory, including `_config.yml`, `index.md`, `_layouts/`, and `explainer-books/`. Use a public repository if you are using GitHub Free.
2. In the repository, open **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Select your publishing branch (usually `main`) and **/ (root)**, then save.
5. Wait for the Pages deployment to succeed. The Pages settings show the published URL. Check the repository's **Actions** tab if the build fails.

GitHub Pages supplies the site's URL and repository prefix automatically. No username or repository name needs to be inserted in the configuration. Do not add a `.nojekyll` file; this site uses Jekyll to render its Markdown and generate the index.

For initial setup, this directory must be initialized as a Git repository, connected to the repository you created, committed, and pushed. Publishing is not enabled just by adding the site files locally.

## Add or update a book

Create a direct subfolder with an overview and any chapters:

```text
explainer-books/
  your-topic/
    00-overview.md
    01-first-chapter.md
    02-second-chapter.md
```

- Start the overview with a descriptive H1, such as `# Your Topic — Explainer Book`. This becomes its home page label, with the ` — Explainer Book` suffix removed.
- Link to chapters with relative Markdown links, such as `[First chapter](./01-first-chapter.md)`.
- Commit and push to the publishing branch. GitHub Pages rebuilds the site, and the new overview appears on the home page automatically in title order.
- Folders without `00-overview.md` and deeper nested folders are not listed. Removing a book folder also removes its index entry on the next deployment.

No front matter or manual home page edits are required for books. Every overview and chapter includes a link back to the library.

## Preview locally

Use Ruby 3.3 and Bundler (rather than macOS's older system Ruby), then run from this directory:

```sh
bundle install
bundle exec jekyll serve
```

Open `http://localhost:4000`. To check links with a GitHub project repository prefix:

```sh
bundle exec jekyll serve --baseurl /preview
```

Open `http://localhost:4000/preview/`. For a build without starting a server:

```sh
bundle exec jekyll build --strict_front_matter
```

The generated site is written to `_site/`, which is ignored by Git. Source books remain plain Markdown. Project documentation and BMAD tooling are excluded from the published site.

After previewing or publishing, check that the home page lists one link per direct book folder with an overview, each overview's chapter links open, and both navigation links on a chapter work. Repeat under `/preview/` to check repository-prefix URLs. Adding and pushing another direct book folder with `00-overview.md` should add its link after the next build.
