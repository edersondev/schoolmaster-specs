Basic

   Field               Required    Description
  ━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   inep_code                yes    Official INEP school code with 8 digits.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   status                   yes    1 active, 0 inactive.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   name                     yes    School display name.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   trade_name                no    Commercial/common name.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   legal_name                no    Registered legal name.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   document       yes on create    CNPJ, digits only in payload. Read-only on edit.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   email                    yes    School contact email, validated as email.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   phone                     no    Contact phone, digits only in payload.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   website                   no    School website URL.
  ─────────────  ───────────────  ──────────────────────────────────────────────────
   description               no    Free text school description.

Institutional

   Field                      Required    Description
  ━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   administrative_type_id          yes    Administrative type lookup ID.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   legal_nature_id                 yes    Legal nature lookup ID.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   management_type_id              yes    Management type lookup ID.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   pedagogical_approach_id         yes    Pedagogical approach lookup ID.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   education_level_ids             yes    Array of education level IDs.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   modality_ids                    yes    Array of modality IDs.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   timezone                         no    Timezone, default America/Sao_Paulo.
  ─────────────────────────  ──────────  ──────────────────────────────────────
   language                         no    Locale/language, default pt-BR.

Institutional group fields + options.

   Field                      Options
  ━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   administrative_type_id     1 Public, 2 Private
  ─────────────────────────  ────────────────────────────────────────────────────────────────────────────────────────────────────────────
   legal_nature_id            1 For Profit, 2 Non Profit
  ─────────────────────────  ────────────────────────────────────────────────────────────────────────────────────────────────────────────
   management_type_id         1 Traditional, 2 Confessional, 3 Community, 4 Philanthropic, 5 Military, 6 Corporate
  ─────────────────────────  ────────────────────────────────────────────────────────────────────────────────────────────────────────────
   pedagogical_approach_id    1 Traditional, 2 Constructivist, 3 Montessori, 4 Waldorf, 5 Bilingual
  ─────────────────────────  ────────────────────────────────────────────────────────────────────────────────────────────────────────────
   education_level_ids        1 Early Childhood Education, 2 Elementary School, 3 High School, 4 Technical Education, 5 Higher Education
  ─────────────────────────  ────────────────────────────────────────────────────────────────────────────────────────────────────────────
   modality_ids               1 On-site, 2 Distance Learning, 3 Hybrid

Branding

   Field              Required    Description
  ━━━━━━━━━━━━━━━━━  ━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   logo_path                no    Backend-managed logo file/path; sent as null when blank.
   logo_url                 no    Browser-ready logo URL returned by backend; null when blank.
  ─────────────────  ──────────  ──────────────────────────────────────────
   primary_color            no    Main brand color, default #1D4ED8.
  ─────────────────  ──────────  ──────────────────────────────────────────
   secondary_color          no    Secondary brand color, default #F59E0B.
