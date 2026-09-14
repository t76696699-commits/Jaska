
# Suhbat xotirasi bilan RAG: chat tarixini saqlash, savolni qayta yozish
# (query rewriting) va 8-darsdagi rag_answer() bilan birlashtirish.

from __future__ import annotations
from dataclasses import dataclass, field

from app.services.grok_ai_client import call_chain, ProviderError


@dataclass(frozen=True)
class ChatTurn:
    """Bitta savol-javob juftligi — immutable (o'zgarmas), 2-darsdagi
    kabi yangi holatni MUTATSIYA qilish o'rniga yangi nusxa yaratamiz."""
    question: str
    answer: str


@dataclass(frozen=True)
class ConversationMemory:
    """Suhbat xotirasi — faqat OXIRGI max_turns juftlikni saqlaydi.
    Immutable: add_turn() joriy obyektni o'zgartirmaydi, YANGI nusxa
    qaytaradi."""
    turns: tuple[ChatTurn, ...] = field(default_factory=tuple)
    max_turns: int = 5

    def add_turn(self, question: str, answer: str) -> "ConversationMemory":
        new_turns = (*self.turns, ChatTurn(question, answer))
        if len(new_turns) > self.max_turns:
            new_turns = new_turns[-self.max_turns:]  # eng eskisini "unutish"
        return ConversationMemory(turns=new_turns, max_turns=self.max_turns)

    def as_history_text(self) -> str:
        if not self.turns:
            return "(hozircha suhbat tarixi yo'q)"
        return "\n".join(f"Savol: {t.question}\nJavob: {t.answer}" for t in self.turns)


async def rewrite_query(new_question: str, memory: ConversationMemory) -> str:
    """Chat tarixidan foydalanib, "uning", "u" kabi ishoralarni
    to'liq, mustaqil savolga aylantiradi. Agar tarix bo'sh bo'lsa,
    LLM'ni chaqirmasdan savolni o'zgarishsiz qaytaradi (keraksiz
    chaqiruvdan saqlanish)."""
    if not memory.turns:
        return new_question

    prompt = (
        "Quyidagi suhbat tarixidan foydalanib, OXIRGI savolni to'liq, "
        "mustaqil (tarixsiz ham tushunarli) savolga qayta yoz. Faqat "
        "qayta yozilgan savolni qaytar, boshqa hech narsa yozma.\n\n"
        f"TARIX:\n{memory.as_history_text()}\n\n"
        f"OXIRGI SAVOL: {new_question}"
    )
    rewritten, _, _, _ = await call_chain(prompt, max_tokens=100)
    return rewritten.strip() or new_question


async def chat_with_memory(new_question: str, memory: ConversationMemory, index: list[dict]) -> tuple[str, ConversationMemory]:
    """To'liq oqim: savolni qayta yozish -> retrieve -> augment ->
    generate -> xotirani yangilash (yangi, immutable nusxa qaytariladi)."""
    from math import sqrt  # noqa: F401 — retrieve() 8-darsdagidek ishlatiladi deb faraz qilinadi

    full_question = await rewrite_query(new_question, memory)

    # 8-darsdagi retrieve/build_augmented_prompt shu yerda chaqiriladi deb faraz qilamiz:
    # chunks = retrieve(full_question, index)
    # prompt = build_augmented_prompt(full_question, chunks) + memory.as_history_text()
    prompt = f"TARIX:\n{memory.as_history_text()}\n\nSAVOL: {full_question}"

    try:
        answer, _, _, _ = await call_chain(prompt, max_tokens=400)
    except ProviderError as e:
        answer = f"AI xizmati vaqtincha ishlamayapti: {e}"

    new_memory = memory.add_turn(new_question, answer)
    return answer, new_memory
