// ════════════════════════════════════════════════════════════════════
// 6-BOSQICH: Testing (Jest + React Testing Library)
// ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) IssueCard.test.tsx - mock Issue turi bilan (TO'G'RI)
// ─────────────────────────────────────────────────────────────────────

import { render, screen } from '@testing-library/react';
import IssueCard from './IssueCard';
import { Issue } from '../../../shared/types';

const mockIssue: Issue = {
  id: 1,
  title: 'Login sahifasi buzilgan',
  description: 'Parolni tiklash tugmasi ishlamayapti',
  status: 'open',
  assigneeId: 7,
  reporterId: 2,
  createdAt: '2026-01-01T10:00:00Z',
};

test("IssueCard sarlavha va holatni ko'rsatadi", () => {
  render(<IssueCard {...mockIssue} />);
  expect(screen.getByText('Login sahifasi buzilgan')).toBeInTheDocument();
  expect(screen.getByText('open')).toBeInTheDocument();
});

// ─────────────────────────────────────────────────────────────────────
// 2) issuesSlice.test.ts - async thunk, fetch mock qilingan
// ─────────────────────────────────────────────────────────────────────

import { configureStore } from '@reduxjs/toolkit';
import issuesReducer, { fetchIssues } from './issuesSlice';

test('fetchIssues muvaffaqiyatli holatni yangilaydi', async () => {
  const mockData: Issue[] = [
    { id: 1, title: 'Test', description: '...', status: 'open',
      assigneeId: null, reporterId: 1, createdAt: '2026-01-01T00:00:00Z' },
  ];
  global.fetch = jest.fn(() =>
    Promise.resolve({ ok: true, json: () => Promise.resolve(mockData) })
  ) as jest.Mock;

  const store = configureStore({ reducer: { issues: issuesReducer } });
  await store.dispatch(fetchIssues());

  expect(store.getState().issues.list).toEqual(mockData);
});

// ─────────────────────────────────────────────────────────────────────
// 3) Ataylab xato - mock "as any" bilan (izohda)
// ─────────────────────────────────────────────────────────────────────

// const mockIssue = {
//   id: 1, title: 'Login sahifasi buzilgan', status: 'open', assigneeId: 7,
// } as any;   // BUTUN tur tekshiruvini o'chiradi!
//
// Issue interfeysi 5-bosqichdagi kabi o'zgarsa ham, bu test HAMON
// YASHIL turaveradi - yolg'on ishonch beradi.
