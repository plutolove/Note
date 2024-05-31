手写递归下降 Parser 时常常会遇到左递归的问题。传统方法是改写文法消除左递归，但需要引入额外的文法，不便于代码的维护。另外，不同的运算符的优先级、结合性都不同。种种原因导致传统递归下降 Parser 的实现难度较大。Pratt Parser算法可以解决这种情况
##### 前置知识
* 前缀运算(Prefix Operator),例如: 负数符号, !取反
    prefix (正规术语是nud)。如果一个token可以放在表达式的最开头，那么它就是一个"prefix"。比如 `123` 、`(`，或者表示负数的`-`。以这种token为中心，构建表达式节点时，不需要知道这个token左边的表达式
* 中缀运算(Infix Operator),例如: + - * / 
	infix (正规术语是led)。如果一个token在构建表达式节点的时候，**必须知道它左边的子表达式**，那么它就是一个"infix"。这意味着infix不能放在任何表达式的开头。
##### Pratt Parser
* 统一定义优先级如下: 
```cpp
enum class Precedence {
  None = 0,
  Assig,
  Or,
  And,
  EQUALITY,
  COMPARISON,
  TERM,
  FACTOR,
  UNARY,
  CALL,
  PRIMARY
};
```

* 给每个token一个优先级,没有设置默认为None
```cpp
static inline std::unordered_map<TokenType, Precedence> type2prece{
    {TokenType::LEFT_PAREN, Precedence::CALL},
    {TokenType::DOT, Precedence::CALL},
    {TokenType::MINUS, Precedence::TERM},
    {TokenType::PLUS, Precedence::TERM},
    {TokenType::STAR, Precedence::FACTOR},
    {TokenType::SLASH, Precedence::FACTOR},
    {TokenType::BANG_EQUAL, Precedence::EQUALITY},
    {TokenType::EQUAL_EQUAL, Precedence::EQUALITY},
    {TokenType::GREATER, Precedence::COMPARISON},
    {TokenType::GREATER_EQUAL, Precedence::COMPARISON},
    {TokenType::LESS, Precedence::COMPARISON},
    {TokenType::LESS_EQUAL, Precedence::COMPARISON},
    {TokenType::AND, Precedence::And},
    {TokenType::OR, Precedence::Or},
    {TokenType::LEFT_BRACKET, Precedence::PRIMARY},
};
```
* 核心解析逻辑如下:
	* 首先解析prefix表达式
	* 然后解析infix表达式,根据优先级判断递归继续向右解析还是立即解析当前表达式例如
	* 表达式: -1+2\*3/4,最开始调用parsePrecedence传入优先级为**Assig**
		* 优先级顺序: (数字) **<**  (- 和+)  **<** (\* 和/)
		* 首先解析 (-1)
		* 由于prece=**Assig**, right_prece=**TERM**, 然后解析+表达式
		* +表达式递归的解析到2,然后调用prefix解析2, 这里继续往后解析,没有直接返回+表达式
		* 由于prece=**TERM**, right_prece=**FACTOR**继续解析/表达式, 这里也没有直接返回\*表达式
		* 然后解析3, 由于prece=**FACTOR**, right_prece=**FACTOR**, 这里没有继续解析/表达式,直接返回\*表达式得到 (* 2 3)
		* 回到\*表达式那一层,继续执行while循环内的解析, 继续向后解析,得到\表达式(/ (\* 2 3) 4)
		* 返回到+那一层得到(+ (-1) (/ (\* 2 3) 4))
```cpp
Maybe<Expr> Parser::parsePrecedence(Precedence prece) {
  auto token = advance();
  auto rule = GET_CLASS(PrefixRuleBase, TokenType, token.type);
  if (not rule) {
    return NewErr("Expect expression, get {}", token.toString());
  }
  auto lhs = JUST(rule->parse(this, token));
  // WARN("lhs str: {}", lhs->to_string());
  auto right_prece = peek().precedence;

  while (prece < right_prece and not isAtEnd()) {
    auto token = advance();
    auto rule = GET_CLASS(InfixRuleBase, TokenType, token.type);
    if (not rule) {
      return NewErr("need binary operator: {}", token.toString());
    }
    lhs = JUST(rule->parse(this, lhs, token));
    WARN("-lhs: {}", lhs->to_string());
    right_prece = peek().precedence;
    // INFO("before: {}, after: {}", token.toString(), peek().toString());
  }
  return lhs;
}
```
* 具体的解析rule实现例子如下
```cpp
Expr UnaryParseRule::parse(Parser* parser, Token token) const {
  auto expr = JUST(parser->parsePrecedence(token.precedence));
  ExprPtr ret = std::make_shared<Unary>(token, expr);
  return ret;
}

Expr BinaryParseRule::parse(Parser* parser, ExprPtr& lhs, Token token) const {
  auto rhs = JUST(parser->parsePrecedence(token.precedence));
  ExprPtr ret = std::make_shared<Binary>(lhs, token, rhs);
  return ret;
}
```