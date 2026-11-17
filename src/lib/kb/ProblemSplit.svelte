<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import { groupByCause, uniqueById, type Report, type Cause } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let {
    parserIssue,
    hintIssue,
  }: {
    parserIssue: { number: number; url: string };
    hintIssue: { number: number; url: string };
  } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === 4471));
  const groups = $derived(groupByCause(merged, tenantOf));
  const issueFor = (cause: Cause | 'unplaced') =>
    cause === 'parser' ? parserIssue : cause === 'hint' ? hintIssue : null;
  const explanation: Record<Cause | 'unplaced', string> = {
    parser: 'The shared parser reads month first, while these tenants write day first.',
    hint: 'The booking-v3 form shows a month-first hint to people in day-first locales.',
    unplaced: 'The export has no tenant configuration to assign a cause.',
  };
  const tags = (group: { taggedAs: Record<string, number> }) =>
    Object.entries(group.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, count]) => `${tag} ×${count}`)
      .join(' · ');
</script>

<p class="summary"><strong>{merged.length}</strong> unique reports merged into #4471 · <strong>{groups.length}</strong> causes</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tenants</th><th>intake tags</th><th>what differs</th><th>issue</th></tr>
  </thead>
  <tbody>
    {#each groups as group (group.cause)}
      {@const issue = issueFor(group.cause)}
      <tr>
        <td><code>{group.cause}</code></td>
        <td>{group.reports.length}</td>
        <td>{group.tenants}</td>
        <td>{tags(group)}</td>
        <td>{explanation[group.cause]}</td>
        <td>{#if issue}<a href={issue.url}>#{issue.number}</a>{:else}unassigned{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .summary { font-size: 1.1rem; }
  td:nth-child(4) { color: var(--muted); font-size: 0.85em; }
</style>
