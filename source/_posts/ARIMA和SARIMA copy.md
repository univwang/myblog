---
title: LCP 69. Hello LeetCode!
date: 2022-10-13 22:42:18
tags: [Leetcode]
excerpt: 一道状态压缩dp的困难题
categories: Leetcode
index_img: /img/index_img/7.png
banner_img: /img/banner_img/background7.jpg
---


```c++
// 二进制位代表个字母所需要位数
// d c t h    o     l    e
// 1 1 1 1  10  11  100

struct  STATE {
	int pos;
	int limit;
	int mask;
	STATE(int p, int l, int m) :pos(p), limit(l), mask(m) {}
	STATE() :pos(0), limit(0), mask(0) {}
};

unordered_map<char, STATE> RULES = {
	// word, {pos, limit, mask}
	{'e', {0, 4, 7}},
	{'l' , {3, 3, 3}},
	{'o' , {5, 2, 3} },
	{'h' , {7, 1, 1}},
	{'t' , {8, 1, 1}},
	{'c' , {9, 1, 1}},
	{'d' , {10, 1, 1}}
};
static constexpr int FULL = 2012;
const int INF = 0x3f3f3f3f;

class Solution {
	// 预处理，计算每个单词的各种情况
	void dfs(string s, int mask, int total, unordered_map<int, int>& cost) {
		if (cost.find(mask) == cost.end() || total < cost[mask]) {
			cost[mask] = total;
		}
		if (s.empty()) {
			return;
		}
		int n = s.size();
		for (int i = 0; i < n; i++) {
			if (RULES.find(s[i]) == RULES.end()) {
				continue;
			}
			STATE& t = RULES[s[i]];
			int cnt = (mask >> t.pos) & t.mask;
			if (cnt < t.limit) {
				dfs(s.substr(0, i) + s.substr(i + 1),
					mask + (1 << t.pos), total + i * (n - 1 - i), cost);
			}
		}
	};

	int merge(int cur, int add) {
		for (const auto& [_, p] : RULES) {
			int c1 = (cur >> p.pos) & p.mask;
			int c2 = (add >> p.pos) & p.mask;
			if (c1 + c2 > p.limit) {
				return -1;
			}
			cur += (c2 << p.pos);
		}
		return cur;
	};

public:
	int Leetcode(vector<string>& words) {
		int n = words.size();

		// 每个单词的 mask, cost
		vector<unordered_map<int, int>> costs;
		for (int i = 0; i < words.size(); i++) {
			unordered_map<int, int> cost;
			dfs(words[i], 0, 0, cost);
			costs.push_back(cost);
		}

		vector<vector<int>> mem(n, vector<int>(1 << 11, -1));
		function<int(int, int)> leetcode = [&](int i, int mask) -> int {
			if (i == n) {
				if (mask == FULL) {
					return 0;
				}
				return INF;    // 不合法
			}
			if (mem[i][mask] != -1) {
				return mem[i][mask];
			}
            int res = INF;
			auto cost = costs[i];
			for (auto& [add, total] : cost) {
				int m = merge(mask, add);
				if (m >= 0) {
					res = min(res, total + leetcode(i + 1, m));
				}
			}
			return mem[i][mask] = res;
		};

		int ans = leetcode(0, 0);
		return ans == INF ? -1 : ans;
	}
};
```