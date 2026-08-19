# Solar Business Philippines Public Media Assets

Public, publication-ready media used by the Solar Business Philippines social scheduling workflow.

## Structure

- `posts/YYYY/MM/` - final images and videos referenced by Metricool
- `asset-manifest.json` - stable mapping from post IDs to public media URLs
- `metricool-schedule.csv` - Metricool-compatible batch scheduling sheet
- `docs/REUSABLE-SOCIAL-PAGE-PUBLISHING-WORKFLOW.md` - complete reusable workflow for other pages and niches

All files in this repository are intentionally public. Draft source files, credentials, private documents, and customer data must never be added.

## Workflow

For the full process, including page configuration, content planning, image production, automated hosting, Metricool scheduling, validation, recovery, and AI-agent handoff, read the [Reusable Social Page Content and Publishing Workflow](docs/REUSABLE-SOCIAL-PAGE-PUBLISHING-WORKFLOW.md).

1. Generate and review a final social asset.
2. Add it under the appropriate dated `posts/` directory.
3. Push the asset to `main`.
4. Use its `raw.githubusercontent.com` URL in Metricool MCP or the batch CSV importer.
5. Verify the scheduled entry in Metricool before publication.
