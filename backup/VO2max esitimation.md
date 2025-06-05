To estimate your maximal oxygen uptake (VO₂ max) from routine runs, you’ll need to:

1. **Estimate the oxygen cost (VO₂) of your running pace**,
2. **Determine your maximum heart rate (HRₘₐₓ)** (or measure it directly in a truly maximal effort), and
3. **Extrapolate your VO₂ max** from the submaximal data.

Below is a step‑by‑step recipe using the three runs you logged.

------

## 1. Convert pace to speed in m/min

| date       | km/h       | → m/min                              |
| ---------- | ---------- | ------------------------------------ |
| 2019-01‑28 | 9.141 km/h | 9.141 × 1000 ÷ 60 ≈ **152.35 m/min** |
| 2019-01‑29 | 8.132 km/h | 8.132 × 1000 ÷ 60 ≈ **135.53 m/min** |
| 2019-02‑07 | 8.602 km/h | 8.602 × 1000 ÷ 60 ≈ **143.37 m/min** |



------

## 2. Estimate the oxygen cost (VO₂) at that pace

For level ground, the ACSM running‐economy equation applies:
$$
\dot V O_{2,\text{ex}} \;(\mathrm{ml/kg/min})
= 0.2 \times \text{speed (m/min)} + 3.5
$$

| date       | speed (m/min) | VO₂_ex (ml/kg/min)          |
| ---------- | ------------- | --------------------------- |
| 2019-01‑28 | 152.35        | 0.2·152.35 + 3.5 ≈ **33.0** |
| 2019-01‑29 | 135.53        | 0.2·135.53 + 3.5 ≈ **30.6** |
| 2019-02‑07 | 143.37        | 0.2·143.37 + 3.5 ≈ **32.2** |



------

## 3. Estimate your HRₘₐₓ

If you haven’t performed an all‐out test, use one of the age‑based formulas as an approximation.
 A common one is Tanaka et al.:
$$
\text{HR}_{\max} \approx 208 - 0.7 \times \text{age}
$$
*(If you know your true maximal heart rate from a field or laboratory test, use that.)*

------

## 4. Extrapolate VO₂ max from submaximal effort

Under the linear assumption between heart rate and VO₂:
$$
\dot V O_{2,\max}
\;=\;
\dot V O_{2,\text{ex}}
\times
\frac{\text{HR}_{\max}}{\text{HR}_{\text{ex}}}
$$
Where:

- $\dot V O_{2,\text{ex}}$ is from step 2,
- $\text{HR}_{\text{ex}}$ is the **average** heart rate you recorded,
- $\text{HR}_{\max}$ is from step 3.

------

### Example calculation

Let’s say you’re 30 years old. Then
$$
\text{HR}_{\max} \approx 208 - 0.7\times30 = 187\ \text{bpm}.
$$
For your 2019-01‑28 run (VO₂_ex ≈ 33.0 ml/kg/min, HR_ex ≈ 167.9 bpm):
$$
\dot V O_{2,\max}
= 33.0 \times \frac{187}{167.9}
\approx 33.0 \times 1.114
\approx \mathbf{36.8\ ml/kg/min}.
$$
You would repeat that for each run:

| date       | VO₂_ex | HR_ex (bpm) | HRₘₐₓ (187) | VO₂_max (ml/kg/min)         |
| ---------- | ------ | ----------- | ----------- | --------------------------- |
| 2019-01‑28 | 33.0   | 167.9       | 187         | 33.0 × 187/167.9 ≈ **36.8** |
| 2019-01‑29 | 30.6   | 157.1       | 187         | 30.6 × 187/157.1 ≈ **36.4** |
| 2019-02‑07 | 32.2   | 161.6       | 187         | 32.2 × 187/161.6 ≈ **37.3** |



Averaging these gives you a VO₂ max in the neighborhood of **36–37 ml/kg/min**.

------

## 5. Alternative: Uth–Sørensen submaximal test

If you also know your **resting** heart rate (HR_rest), you can use the simpler Uth formula:
$$
\dot V O_{2,\max}
\;=\;
15.3 \times \frac{\text{HR}_{\max}}{\text{HR}_{\text{rest}}}
$$
This bypasses the need to estimate VO₂_ex from pace, but **does** require an accurate resting heart rate (measured in supine rest, ideally first thing in the morning).

------

### Caveats & Tips

- **Accuracy** hinges on**:
  - HRₘₐₓ** being close to your true maximum.
  - The linear HR–VO₂ relationship (valid ~50–85% of VO₂ max).
  - A perfectly level course (no hills).
- **Field vs. lab:** A maximal graded exercise test in a lab remains the gold standard; these submaximal estimates usually carry an error of ±5–10%.
- **Practical use:** Track your estimates over weeks; a rising VO₂ max suggests improving fitness (even if the absolute number is slightly off).

------

**Bottom line:**

1. Compute your running VO₂ at submax pace using the ACSM equation.
2. Estimate or measure your HRₘₐₓ (and maybe HR_rest).
3. Extrapolate to VO₂ max via the HR ratio (or Uth formula).

That workflow will yield a reasonable VO₂ max estimate from routine GPS + HR runs.
![Image](https://github.com/user-attachments/assets/0808198e-7a10-43da-8449-c8cc1d6d5a5b)