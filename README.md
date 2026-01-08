"""
LRU Cache Implementation
------------------------
A production-grade Least Recently Used cache.

Features:
- O(1) get and put
- Doubly linked list + hash map
- Clean and deterministic eviction
- Used in real systems (databases, OS, browsers)

Run:
python lru_cache.py
"""

from typing import Optional


class Node:
    def __init__(self, key: int, value: int):
        self.key = key
        self.value = value
        self.prev: Optional["Node"] = None
        self.next: Optional["Node"] = None


class LRUCache:
    def __init__(self, capacity: int):
        if capacity <= 0:
            raise ValueError("Capacity must be greater than zero")

        self.capacity = capacity
        self.cache = {}

        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head

    # -------------------------
    # Internal helpers
    # -------------------------
    def _remove(self, node: Node):
        prev_node = node.prev
        next_node = node.next
        prev_node.next = next_node
        next_node.prev = prev_node

    def _add_to_front(self, node: Node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    # -------------------------
    # Public API
    # -------------------------
    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1

        node = self.cache[key]
        self._remove(node)
        self._add_to_front(node)
        return node.value

    def put(self, key: int, value: int):
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._remove(node)
            self._add_to_front(node)
            return

        if len(self.cache) >= self.capacity:
            lru = self.tail.prev
            self._remove(lru)
            del self.cache[lru.key]

        new_node = Node(key, value)
        self.cache[key] = new_node
        self._add_to_front(new_node)

    def __repr__(self):
        values = []
        current = self.head.next
        while current != self.tail:
            values.append(f"{current.key}:{current.value}")
            current = current.next
        return " -> ".join(values)


# -------------------------
# Demo
# -------------------------
def main():
    cache = LRUCache(capacity=3)

    cache.put(1, 100)
    cache.put(2, 200)
    cache.put(3, 300)
    print(cache)

    cache.get(1)
    cache.put(4, 400)

    print(cache)
    print("Get 2:", cache.get(2))
    print("Get 1:", cache.get(1))


if __name__ == "__main__":
    main()
