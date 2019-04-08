```python
class Solution:
    def a_u_b(self,a,b):
        res = [i for i in a]
        for j in b:
            if j not in res:
                res.append(j)
        return res
    def a_n_b(self,a,b):
        res = []
        for j in a:
            if j in b:
                res.append(j)
        return res  
    def u_n(self,u,n):
        res = []
        for i in u:
            if i not in n:
                res.append(i)
        return res
    def diff(self,a,b): 
        u = self.a_u_b(a,b)
        n = self.a_n_b(a,b)
        u_n = self.u_n(u,n)
        res = sorted(u_n)
        return res

if __name__ == '__main__':
    solution = Solution()
    a = [1,2,3]
    b = [2,3,4]
    res = solution.diff(a,b)
    print(res)
```