---
title: 'picoCTF 2023 | Rotation WriteUp'
author: appdone
categories: [picoCTF 2023 Challenges]
render_with_liquid: false
---

Bu yazıda, picoCTF platformunda yer alan "Rotation" isimli meydan okumayı çözeceğiz. ROT13 ile kodlanmış gibi görünen bir metin duruyor karşımızda.

```
xqkwKBN{z0bib1wv_l3kzgxb3l_7l140864}
```

Metni Chepy aracına girdikten sonra "rotate_bruteforce" komutunu kullandığımızda teker teker deniyor ve bize sonuçları gösteriyor.

```console
$ chepy "xqkwKBN{z0bib1wv_l3kzgxb3l_7l140864}"
>>> rotate_bruteforce
{'1': b'yrlxLCO{a0cjc1xw_m3lahyc3m_7m140864}', '2': b'zsmyMDP{b0dkd1yx_n3mbizd3n_7n140864}', '3': b'atnzNEQ{c0ele1zy_o3ncjae3o_7o140864}',
'4': b'buoaOFR{d0fmf1az_p3odkbf3p_7p140864}', '5': b'cvpbPGS{e0gng1ba_q3pelcg3q_7q140864}', '6': b'dwqcQHT{f0hoh1cb_r3qfmdh3r_7r140864}',
'7': b'exrdRIU{g0ipi1dc_s3rgnei3s_7s140864}', '8': b'fyseSJV{h0jqj1ed_t3shofj3t_7t140864}', '9': b'gztfTKW{i0krk1fe_u3tipgk3u_7u140864}',
'10': b'haugULX{j0lsl1gf_v3ujqhl3v_7v140864}', '11': b'ibvhVMY{k0mtm1hg_w3vkrim3w_7w140864}', '12': b'jcwiWNZ{l0nun1ih_x3wlsjn3x_7x140864}',
'13': b'kdxjXOA{m0ovo1ji_y3xmtko3y_7y140864}', '14': b'leykYPB{n0pwp1kj_z3ynulp3z_7z140864}', '15': b'mfzlZQC{o0qxq1lk_a3zovmq3a_7a140864}',
'16': b'ngamARD{p0ryr1ml_b3apwnr3b_7b140864}', '17': b'ohbnBSE{q0szs1nm_c3bqxos3c_7c140864}', '18': b'picoCTF{***}',
'19': b'qjdpDUG{s0ubu1po_e3dszqu3e_7e140864}', '20': b'rkeqEVH{t0vcv1qp_f3etarv3f_7f140864}', '21': b'slfrFWI{u0wdw1rq_g3fubsw3g_7g140864}',
'22': b'tmgsGXJ{v0xex1sr_h3gvctx3h_7h140864}', '23': b'unhtHYK{w0yfy1ts_i3hwduy3i_7i140864}', '24': b'voiuIZL{x0zgz1ut_j3ixevz3j_7j140864}',
'25': b'wpjvJAM{y0aha1vu_k3jyfwa3k_7k140864}', '26': b'xqkwKBN{z0bib1wv_l3kzgxb3l_7l140864}'}
>>>
```
