// ════════════════════════════════════════════════════════════════════
// 5-BOSQICH: React frontend + Redux Toolkit (TypeScript)
// ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) fetchJson<T> - tekshiruvsiz assertion (2-darsdagi kabi tanish naqsh)
// ─────────────────────────────────────────────────────────────────────

export async function fetchJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`So'rov xato: ${res.status}`);
  return res.json() as Promise<T>;
}

// ─────────────────────────────────────────────────────────────────────
// 2) issuesSlice.ts - createAsyncThunk + shared Issue turi
// ─────────────────────────────────────────────────────────────────────

import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { Issue } from '../../../shared/types';

export const fetchIssues = createAsyncThunk('issues/fetch', async () => {
  return fetchJson<Issue[]>('/api/issues');
});

const issuesSlice = createSlice({
  name: 'issues',
  initialState: { list: [] as Issue[], status: 'idle' as 'idle' | 'loading' | 'succeeded' | 'failed' },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchIssues.pending, (state) => { state.status = 'loading'; })
      .addCase(fetchIssues.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.list = action.payload;
      });
  },
});

export default issuesSlice.reducer;

// ─────────────────────────────────────────────────────────────────────
// 3) store/hooks.ts - tiplashtirilgan hook'lar
// ─────────────────────────────────────────────────────────────────────

import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// ─────────────────────────────────────────────────────────────────────
// 4) IssueCard.tsx - Pick<Issue, ...> orqali props (izohda - JSX)
// ─────────────────────────────────────────────────────────────────────

// type IssueCardProps = Pick<Issue, 'id' | 'title' | 'status' | 'assigneeId'>;
//
// function IssueCard({ id, title, status, assigneeId }: IssueCardProps) {
//   return (
//     <div className="issue-card">
//       <h4>{title}</h4>
//       <span>{status}</span>
//       <p>{assigneeId ? `Tayinlangan: #${assigneeId}` : 'Tayinlanmagan'}</p>
//     </div>
//   );
// }

// ─────────────────────────────────────────────────────────────────────
// 5) Ataylab xato - backend maydonni o'zgartiradi, shared/types.ts eski qoladi (izohda)
// ─────────────────────────────────────────────────────────────────────

// Backend YANGI javob: { ..., "assignee": { "id": 7, "name": "Aziz" } }
// shared/types.ts ESKI: assigneeId: number | null  (yangilanmagan!)
// Natija: issue.assigneeId HAR DOIM undefined - lekin xato yo'q, crash yo'q,
// faqat UI har doim "Tayinlanmagan" deb ko'rsatadi.
