=== "Unicode"

    ``` yaml
    markdown_extensions:
      - toc:
          slugify: !!python/object/apply:pymdownx.slugs.slugify
            kwds:
              case: lower
    ```

=== "Unicode, case-sensitive"

    ``` yaml
    markdown_extensions:
      - toc:
          slugify: !!python/object/apply:pymdownx.slugs.slugify
    ```