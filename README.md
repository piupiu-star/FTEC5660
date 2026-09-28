# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
> to students: please fill your solution description here.

## Homework 1 solution:

### Chain design

![Chain design](chain_design.png)

### Solution description

I built a vision-based LangChain chain using the deepseek-v4-flash-vision-exp model to extract the monetary fields from supermarket receipt images and answer the two given queries. As sketched in the diagram above, the overall design consists of four stages: image encoding, structured extraction, aggregation in Python, and response-format control.
Specifically, each receipt image is first converted into a base64-encoded data URL with the provided ‘image_data_url(path)’ helper, a representation that multimodal models accept directly. The model is then instructed to return nothing but a structured JSON object with three fields: ‘final_payment’ (the final amount paid after the ROUNDING line),’ subtotal; (the amount on the SUBTOTAL line), and ‘discount_lines ‘(an array of every discount, promotion, and coupon amount, each reported as a positive number). The chain itself is composed with LangChain's LCEL syntax (prompt | model), with the model configured at temperature=0.2 and max_tokens=2048. In ‘answer_queries’, the receipts are processed one by one via ‘chain.invoke’, retrying up to three times whenever the model returns an empty response. And the aggregation is carried out in Python with Decimal arithmetic: Query 1 = Σ final_payment, and Query 2 = Σ (subtotal + Σ discount_lines). This design deliberately avoids asking the model to perform arithmetic, which is unreliable, and leaves all computation to Python. The final responses are formatted as HK$xxxx.xx, each containing exactly one number.
